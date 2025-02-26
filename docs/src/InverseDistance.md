The **Inverse Distance Surrogate** is an interpolating method and in this method the unknown points are calculated with a weighted average of the sampling points. This model uses the inverse distance between the unknown and training points to predict the unknown point. We do not need to fit this model because the response of an unknown point x is computed with respect to the distance between x and the training points.

Let's optimize the following function to use Inverse Distance Surrogate:

$f(x) = sin(x) + sin(x)^2 + sin(x)^3$.

First of all, we have to import these two packages: `Surrogates` and `Plots`.

```@example Inverse_Distance1D
using Surrogates
using Plots
default()
```


### Sampling

We choose to sample f in 25 points between 0 and 10 using the `sample` function. The sampling points are chosen using a Low Discrepancy, this can be done by passing `LowDiscrepancySample()` to the `sample` function.

```@example Inverse_Distance1D
f(x) = sin(x) + sin(x)^2 + sin(x)^3

n_samples = 25
lower_bound = 0.0
upper_bound = 10.0
x = sample(n_samples, lower_bound, upper_bound, LowDiscrepancySample(;base=2))
y = f.(x)

scatter(x, y, label="Sampled points", xlims=(lower_bound, upper_bound), legend=:top)
plot!(f, label="True function", xlims=(lower_bound, upper_bound), legend=:top)
```


## Building a Surrogate

```@example Inverse_Distance1D
InverseDistance = InverseDistanceSurrogate(x, y, lower_bound, upper_bound)
add_point!(InverseDistance, 5.0, f(5.0))
add_point!(InverseDistance, [5.1,5.2], [f(5.1),f(5.2)])
prediction = InverseDistance(5.0)
```

Now, we will simply plot `InverseDistance`:

```@example Inverse_Distance1D
plot(x, y, seriestype=:scatter, label="Sampled points", xlims=(lower_bound, upper_bound), legend=:top)
plot!(f, label="True function",  xlims=(lower_bound, upper_bound), legend=:top)
plot!(InverseDistance, label="Surrogate function",  xlims=(lower_bound, upper_bound), legend=:top)
```


## Optimizing

Having built a surrogate, we can now use it to search for minima in our original function `f`.

To optimize using our surrogate we call `surrogate_optimize` method. We choose to use Stochastic RBF as optimization technique and again Sobol sampling as sampling technique.

```@example Inverse_Distance1D
@show surrogate_optimize(f, SRBF(), lower_bound, upper_bound, InverseDistance, SobolSample())
scatter(x, y, label="Sampled points", legend=:top)
plot!(f, label="True function",  xlims=(lower_bound, upper_bound), legend=:top)
plot!(InverseDistance, label="Surrogate function",  xlims=(lower_bound, upper_bound), legend=:top)
```


## Inverse Distance Surrogate Tutorial (ND):

First of all we will define the `Schaffer` function we are going to build surrogate for. Notice, one how its argument is a vector of numbers, one for each coordinate, and its output is a scalar.

```@example Inverse_DistanceND
using Plots # hide
default(c=:matter, legend=false, xlabel="x", ylabel="y") # hide
using Surrogates # hide

function schaffer(x)
    x1=x[1]
    x2=x[2]
    fact1 = (sin(x1^2-x2^2))^2 - 0.5;
    fact2 = (1 + 0.001*(x1^2+x2^2))^2;
    y = 0.5 + fact1/fact2;
end
```


### Sampling

Let's define our bounds, this time we are working in two dimensions. In particular we want our first dimension `x` to have bounds `-5, 10`, and `0, 15` for the second dimension. We are taking 60 samples of the space using Sobol Sequences. We then evaluate our function on all of the sampling points.

```@example Inverse_DistanceND
n_samples = 60
lower_bound = [-5.0, 0.0]
upper_bound = [10.0, 15.0]

xys = sample(n_samples, lower_bound, upper_bound, SobolSample())
zs = schaffer.(xys);
```

```@example Inverse_DistanceND
x, y = -5:10, 0:15 # hide
p1 = surface(x, y, (x1,x2) -> schaffer((x1,x2))) # hide
xs = [xy[1] for xy in xys] # hide
ys = [xy[2] for xy in xys] # hide
scatter!(xs, ys, zs) # hide
p2 = contour(x, y, (x1,x2) -> schaffer((x1,x2))) # hide
scatter!(xs, ys) # hide
plot(p1, p2, title="True function") # hide
```


### Building a surrogate
Using the sampled points we build the surrogate, the steps are analogous to the 1-dimensional case.

```@example Inverse_DistanceND
InverseDistance = InverseDistanceSurrogate(xys, zs,  lower_bound, upper_bound)
```

```@example Inverse_DistanceND
p1 = surface(x, y, (x, y) -> InverseDistance([x y])) # hide
scatter!(xs, ys, zs, marker_z=zs) # hide
p2 = contour(x, y, (x, y) -> InverseDistance([x y])) # hide
scatter!(xs, ys, marker_z=zs) # hide
plot(p1, p2, title="Surrogate") # hide
```


### Optimizing
With our surrogate we can now search for the minima of the function.

Notice how the new sampled points, which were created during the optimization process, are appended to the `xys` array.
This is why its size changes.

```@example Inverse_DistanceND
size(xys)
```
```@example Inverse_DistanceND
surrogate_optimize(schaffer, SRBF(), lower_bound, upper_bound, InverseDistance, SobolSample(), maxiters=10)
```
```@example Inverse_DistanceND
size(xys)
```

```@example Inverse_DistanceND
p1 = surface(x, y, (x, y) -> InverseDistance([x y])) # hide
xs = [xy[1] for xy in xys] # hide
ys = [xy[2] for xy in xys] # hide
zs = schaffer.(xys) # hide
scatter!(xs, ys, zs, marker_z=zs) # hide
p2 = contour(x, y, (x, y) -> InverseDistance([x y])) # hide
scatter!(xs, ys, marker_z=zs) # hide
plot(p1, p2) # hide
```
