# PS_con: Shape-Constrained Penalized Splines
Penalized splines is a popular method for function estimation under the assumption of smoothness.
The provided code extends the well known P-spline method to constraints on its shape, such as monotonicity or convexity, and allows for the consideration of additional random effects within the model.
Further detail can be found in [Wagner et al., 2017](https://rss.onlinelibrary.wiley.com/doi/full/10.1111/rssa.12295).

## Manual
For selected parameters, the B-spline basis matrix and the matrix of the penalty term are assembled, where the difference penalty or the curvature penalty (default) can be used:
```R
Phi <- bspline_matrix(x, m, q, Omega)     # B-spline basis matrix at the covariates
#Delta <- diff(diag(K), diff=l)           # difference penalty
Delta <- L2_norm_matrix(m, q, l, Omega)   # curvature penalty (default)
```

The coefficients of the common P-spline are determined via the solution of a linear system:
```R
A <- crossprod(Phi) + lambda*crossprod(Delta)    # coefficient matrix
b <- crossprod(Phi,y)                            # right-hand site
alpha <- solve(A,b)                              # coefficient vector as solution of the linear system
```

The matrices for the user-specific shape constraints can be assembled by the functions made available:
```R
C1 <- shape_constraints_matrix(x_grid, m, q, r=0, Omega)      # nonnegativity constraint
C2 <- -shape_constraints_matrix(x_grid, m, q, r=2, Omega)     # concavity constraint
C <- rbind(C1,C2)                                         
```

They are incorporated into the spline fitting process as linear constraints onto the loss-function, leading a quadratic program, which is solved using the `solve.QP` function of the `quadprog` package:
```R
A <- crossprod(Phi) + lambda*crossprod(Delta)             # matrix for the quadratic program 
b <- crossprod(Phi,y)                                     # vector for the quadratic program
sol <- solve.QP(A, b, t(C))                               # solve the quadratic program
alpha <- sol$solution   
```

Further random effects can be taken into account by means of a rondom effects matrix `W` and an respective extension of the above matrices:
```R
W <- diag(D)[area,]                                     # intercept indicator matrix
B <- cbind(Phi, W)                                      # extension of the basis matrix
P <- bdiag(crossprod(Delta), diag(D))                   # extension of the penaly matrix
C_ext <- cbind(C, matrix(0, nrow=dim(C)[1], ncol=D))    # extension of the shape constraint matrix
```

The resulting quadratic program is still solved with `solve.QP`:
```R
A <- crossprod(B) + lambda*P            # matrix for the quadratic program 
b <- crossprod(y, B)                    # matrix for the quadratic program 
sol <- solve.QP(A, b, t(C_ext))         # solve the quadratic program
alpha <- sol$solution[1:K]
```

## Test Results
For a simple test data set
```R
n <- 100                          
x <- sort( runif(n) )
fx <- sin(pi*x)                                                  # arbitrary test function
D <- 10                                                          # number of random intercepts
area <- sample(1:10, n, replace=T)                               # intercept indicator
y <- fx + rnorm(n, sd=0.1) + rnorm(length(area), sd=0.5)[area]   # test function + random error + area specific intercept
```
the estimated functions look like follows:
<img src="https://user-images.githubusercontent.com/46927836/58548243-38286a00-8209-11e9-8a41-98ae5b7dcb47.png" width="80%">


## Application
<img src="https://user-images.githubusercontent.com/46927836/58548240-378fd380-8209-11e9-89bc-4570328d19b6.png" width="80%">
<img src="https://user-images.githubusercontent.com/46927836/58548241-38286a00-8209-11e9-859f-71808a0ad47f.png" width="80%">
<img src="https://user-images.githubusercontent.com/46927836/58548242-38286a00-8209-11e9-8aec-e37c01efb2e8.png" width="80%">

![FunctionFit-1](https://user-images.githubusercontent.com/46927836/58548240-378fd380-8209-11e9-89bc-4570328d19b6.png)
![MAP_estimates-1](https://user-images.githubusercontent.com/46927836/58548241-38286a00-8209-11e9-859f-71808a0ad47f.png)
![MAP_rrmse-1](https://user-images.githubusercontent.com/46927836/58548242-38286a00-8209-11e9-8aec-e37c01efb2e8.png)

