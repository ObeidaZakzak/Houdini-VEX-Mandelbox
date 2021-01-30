# Houdini-VEX-Mandelbox

```C
function float boxFoldAxis(float x)
{
    if (x > 1) return(2-x);
    if (x < -1) return(-2 - x);
    return (x);
}
```

```C
function vector boxFold(vector pt)
{
    float x = boxFoldAxis(pt.x);
    float y = boxFoldAxis(pt.y);
    float z = boxFoldAxis(pt.z);
    
    return(set(x,y,z));
}
```

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

