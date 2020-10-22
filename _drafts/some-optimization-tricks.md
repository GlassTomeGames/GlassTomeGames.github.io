---
layout: post
title:  "A couple math-based micro-optimization tricks"
author: James Lucas
tags: tutorial unity
---

I love micro-optimizing my code with maths. This is rarely necessary, but I dislike unnecessary computation. Adopting these optimization tricks wont double your FPS, but you can sleep happily knowing that you avoided a square root.

<script src="https://polyfill.io/v3/polyfill.min.js?features=es6"></script>
<script id="MathJax-script" async src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js"></script>

All of the code below is written in c#, and uses `UnityEngine`'s data types throughout. But the same ideas can easily be applied elsewhere.

## Distance comparisons

Let's start with an easy one. Suppose we want to find the point in an array that is closest to some position in 3D space.

This optimization shouldn't be too surprising, but I frequently see code that reads like this first snippet.

### Please, no!

{% highlight cs %}
public int FindClosest(Vector3 p, Vector3[] targets)
{
    int min = -1;
    float minDist = float.MaxValue;

    for (int i = 0; i < targets.Length; i++) {
        float dist = Vector3.Distance(p, targets[i]);
        if (dist < minDist)
        {
            min = i;
            minDist = dist;
        }
    }
    return min;
}
{% endhighlight %}

What's wrong with this code? Lets begin by unpacking the `Vector3.Distance` function.

{% highlight cs %}
public int FindClosest(Vector3 p, Vector3[] targets)
{
    //...
        //float dist = Vector3.Distance(p, targets[i]);
        Vector3 diff = p - targets[i];
        float dist = Mathf.Sqrt(Vector3.Dot(diff, diff));
        if (dist < minDist) { /* ... */ }
    //...
}
{% endhighlight %}

Notice that the function output is exactly the same if we replace the distance check with:

{% highlight cs %}
if (dist * dist < minDist * minDist) { /* ... */ }
}
{% endhighlight %}

So we don't need this square root at all!

### More efficient distance checking

{% highlight cs %}
public int FindClosest(Vector3 p, Vector3[] targets)
{
    int min = -1;
    float minSqDist = float.MaxValue;

    for (int i = 0; i < targets.Length; i++) {
        Vector3 diff = p - targets[i];
        float sqDiff = Vector3.Dot(diff, diff);
        if (sqDiff < minSqDist)
        {
            min = i;
            minSqDist = sqDiff;
        }
    }
    return min;
}
{% endhighlight %}

## Field-of-view checking

Now we'll make the previous example a little more interesting. We have an enemy that uses sight to detect the player. They have a sight-range, and a maximum angle that they can see in front of them.

### A first attempt

We now know how to efficiently check the distance. We'll add an angle check:

{% highlight cs %}
/// Check if target is within field of view
///
/// Arguments:
///
///    - pos:           Position of enemy
///    - facingDir:     Direction enemy is facing
///    - playerPos:     Position of player
///    - maxSqDistance: Maximum squared distance of sight
///    - maxAngle:      Maximum viewing angle
public bool CheckFoV(Vector3 pos, Vector3 facingDir, Vector3 playerPos, float maxSqDistance, float maxAngle)
{
    Vector3 diff = playerPos - pos;
    if (Vector3.Dot(diff, diff) > maxSqDistance) {
        // Out of sight range
        return false;
    }

    if (Vector3.Dot(facingDir, diff) < 0) {
        // Target is behind us
        return false;
    }

    if (Vector3.Angle(facingDir, diff) > maxAngle) {
        // Outside of viewing range
        return false;
    }
    return true;
}
{% endhighlight %}

This one looks fairly good. But to see what might be going wrong, we need to dig into [Unity's `Vector3.Angle` function](https://github.com/Unity-Technologies/UnityCsReference/blob/61f92bd79ae862c4465d35270f9d1d57befd1761/Runtime/Export/Math/Vector3.cs#L305-L314).

{% highlight cs %}
/// https://github.com/Unity-Technologies/UnityCsReference/blob/61f92bd79ae862c4465d35270f9d1d57befd1761/Runtime/Export/Math/Vector3.cs#L305-L314
public static float Angle(Vector3 from, Vector3 to)
{
    // sqrt(a) * sqrt(b) = sqrt(a * b) -- valid for real numbers
    float denominator = (float)Math.Sqrt(from.sqrMagnitude * to.sqrMagnitude);
    if (denominator < kEpsilonNormalSqrt)
        return 0F;

    float dot = Mathf.Clamp(Dot(from, to) / denominator, -1F, 1F);
    return ((float)Math.Acos(dot)) * Mathf.Rad2Deg;
}
{% endhighlight %}

There are some red flags here. First, a square root in the first line and then the return value uses an arcosine. That wont do.

### More efficient field-of-view checking

The dot product can equivalently be written,

$$\mathbf{a} \cdot \mathbf{b} = || \mathbf{a} || \: ||\mathbf{b}|| \cos(\theta).$$

We want to check if $$\theta < \theta^*$$, the maximum viewing angle. Note that we rule out obtuse angles in our function, by checking the sign of the dot product.

Knowing that the angle is acute helps us to remove these unwanted operations. Because if $$\theta < \theta^*$$ are both acute, then $$\cos(\theta) > \cos(\theta^*)$$. It follows that,
$$\cos^2(\theta) > \cos^2(\theta^*)$$. And now we don't need the square root either!

Our new function is:

{% highlight cs %}
public bool CheckFoV(Vector3 pos, Vector3 facingDir, Vector3 playerPos, float maxSqDistance, float maxCosSqAngle)
{
    Vector3 diff = playerPos - pos;
    if (Vector3.Dot(diff, diff) > maxSqDistance) {
        // Out of sight range
        return false;
    }
    float angleCheckDot = Vector3.Dot(facingDir, diff);
    if (angleCheckDot < 0) {
        // Target is behind us
        return false;
    }

    float posSqMagnitude = pos.sqrMagnitude;
    float playerPosSqMagnitude = playerPos.sqrMagnitude;
    float cosSqTheta = angleCheckDot * angleCheckDot / (posSqMagnitude * playerPosSqMagnitude);
    if (cosSqTheta < maxCosSqAngle) {
        // Outside of viewing range
        return false;
    }
    return true;
}
{% endhighlight %}

Note that this function takes in the maximum cosine of the angle squared, instead of the angle itself. This is so that we don't need to compute this within the function each time.


## Screen Rotation

We use a similar trick to the above for our screen rotation implementation in Sprawl.

# Why?

Ultimately, the tricks above are unlikely to make a significant difference to your runtime performance. Choosing good data structures, good algorithms, and optimizing cache efficiency are going to be more important for your game's performance.

But if these computations are appearing frequently in your game --- maybe you need to poll a hundred enemies' FoV per frame --- then these tricks could make a big difference. For me though, knowing that I _can_ do something better is more than enough justification to do it. The question then is, why not?
