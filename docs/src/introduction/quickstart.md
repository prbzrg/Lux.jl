# Quickstart

:::tip PRE-REQUISITES

You need to install `Optimisers` and `Zygote` if not done already.
`Pkg.add(["Optimisers", "Zygote"])`

:::

```@example quickstart
using Lux, Random, Optimisers, Zygote
# using LuxCUDA, LuxAMDGPU, Metal # Optional packages for GPU support
```

We take randomness very seriously

```@example quickstart
# Seeding
rng = Random.default_rng()
Random.seed!(rng, 0)
```

Build the model

```@example quickstart
# Construct the layer
model = Chain(BatchNorm(128), Dense(128, 256, tanh), BatchNorm(256),
    Chain(Dense(256, 1, tanh), Dense(1, 10)))
```

Models don't hold parameters and states so initialize them. From there on, we just use our
standard AD and Optimisers API.

```@example quickstart
# Get the device determined by Lux
device = gpu_device()

# Parameter and State Variables
ps, st = Lux.setup(rng, model) .|> device

# Dummy Input
x = rand(rng, Float32, 128, 2) |> device

# Run the model
y, st = Lux.apply(model, x, ps, st)

# Gradients
## Pullback API to capture change in state
(l, st_), pb = pullback(p -> Lux.apply(model, x, p, st), ps)
gs = pb((one.(l), nothing))[1]

# Optimization
st_opt = Optimisers.setup(Adam(0.0001f0), ps)
st_opt, ps = Optimisers.update(st_opt, ps, gs)
```