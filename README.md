# Houdini-VEX-Mandelbox

<img src="https://github.com/ObeidaZakzak/Houdini-VEX-Mandelbox/blob/main/mandelbox_renders/1.png" width="400" height="400"> <img src="https://github.com/ObeidaZakzak/Houdini-VEX-Mandelbox/blob/main/mandelbox_renders/4.png" width="400" height="400">

Check it on [ArtStation](https://www.artstation.com/artwork/Krk3ao)

This is a breakdown of the VEX code used to build the Mandelbox you see in the pictures above. Before starting, I recommend that you check the [Mandelbulb Tutorial from Entagma](https://www.sidefx.com/tutorials/vex-in-houdini-mandelbrot-and-mandelbulb/) to understand better the volume setup. The VEX code below is an adaptation of Entagma's tutorial with extra coding and different operations on the points in order to get a Mandelbox instead of a Mandelbulb.

The main idea is to apply an operation on each point of space several times until it reaches a given max number of iterations and by maintaining the distance between the point and the center of the box under a given limit.
The operation to be applied at each point (x is considered as a point in the formula) : 
<img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/1d9ea0d7b00d8c135f1fdd67727d2834e0dbe58b">


first of all, let's write the `boxFold` operation for a single axis :
```C
function float boxFoldAxis(float x)
{
    if (x > 1) return(2-x);
    if (x < -1) return(-2 - x);
    return (x);
}
```

Now let's write `boxFold` for a given point by applying the `boxfoldAxis` at each axis :

```C
function vector boxFold(vector pt)
{
    float x = boxFoldAxis(pt.x);
    float y = boxFoldAxis(pt.y);
    float z = boxFoldAxis(pt.z);
    
    return(set(x,y,z));
}
```

We need now the `ballFold` operation which modifies the magnitude of a given point :

```C
function vector ballFold(float r; vector pt)
{
    float m = distance(set(0,0,0), pt);
    
    if (m < r) {
        m = m / pow(r,2);
    } else if (m < 1) {
        m = 1 / m;
    }
    return(m * normalize(pt));
}
```

The latest function is the one that iterates and modifies the points until it reaches the maximum number of iterations or the distance limit between the point and the center (0,0,0) :

```C
function int Mandelbox(float s, r, f, L; int imax; vector pt0)
{
    int i;
    vector pt = pt0;
    
    for (i = 0; i < imax; i++)
    {
    
        pt = s*ballFold(r, f*boxFold(pt)) + pt;
        
        if (distance(set(0,0,0), pt) > L)
            return (i);
    }
    return (imax);
}
```

Once we have these functions written, we can now write the script modifying the volume in order to build the fractal form :

```C
int maxiter = chi('MaxIter');

float s = ch('s');
float r = ch('r');
float f = ch('f');
float L = ch('L');

if (Mandelbox(s, r, f, L, maxiter, v@P) < maxiter) {
    @density = 0.0;
} else {
    @density = 1.0;
}
```

