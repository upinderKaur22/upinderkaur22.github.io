---
layout: page
title: Project IV
description: Finite Volume Mimetic Finite Difference Using Staggered Mesh.
img: /assets/img/Project4/Background.png
---

The aim of this project is to solve the incompressible Navier-Stokes equations using the staggered-mesh, second-order finite-volume/mimetic finite-difference method. The problem, we are dealing with this time, is a flow within a rectangular box with aspect ratio $$L_x/L_y$$. The top wall will move with a constant speed $$U$$ to the right, generating the so--called ***lid driven cavity*** flow. 

We will consider, for simplicity, a uniform grid with equal grid spacing, $$h_x$$ in the $$x$$--direction, and $$h_y$$ in the $$y$$--directions. In what follows, the quations are made nondimensional by the Reynold's number, so that the results can be compared with the theoretical ones given in [[2]](#2).

### Incompressible Navier--Stokes equation.

The dimensionless incompressible Navier--Stokes equation in its traditional form for solving the velocity field is given as follows

$$
  \partial_t \textbf{u} + \textbf{u} \cdot \nabla \textbf{u} + \nabla p = \frac{1}{R_e} \, \nabla^2 \textbf{u} \,,
$$

$$
  \nabla \cdot \textbf{u} = 0 \,,
$$

where the quantity $$p(x,y)$$ represents the preassure at point $$(x,y)$$, $$R_e$$ represents the Reynold's number, and $$\textbf{u}$$ the velocity field in two-dimensions which is given by 

$$\textbf{u}(x,y)=\left[ u(x,y), v(x,y) \right]^\top \,,$$ 

on a rectangular domain $$\Omega = \left[0, L_x \right] \times \left[0, L_y \right]$$. 

The four boundaries are denoted $$\textbf{N}$$orth, $$\textbf{S}$$outh, $$\textbf{W}$$est, and $$\textbf{E}$$ast. The domain is fixed in time, and we consider no--slip boundary conditions on each wall. In this regard, the boundary condition impossed in the problem are such that, 

For the upper ($$\textbf{N}$$orth) and lower ($$\textbf{S}$$outh) bounds:

$$
u(x, L_y ) = u_N(x) \,,
$$

$$
v(x, L_y ) = 0 \,,
$$

$$
u(x, 0   ) = u_S(x) \,,
$$

$$
v(x, 0   ) = 0 \,,
$$

For the left ($$\textbf{W}$$est) and right ($$\textbf{E}$$ast) bounds:

$$u(0, y   ) = 0  \,,
$$

$$
v(0, y   ) = v_W(y) \,,
$$

$$
u(L_x , y) = 0 \,,
$$

$$
v(L_x , y) = v_E(y)  \,,
$$

For the development of this project, and in what it follows, we have considered $$v_E(y) = 0$$, $$v_W(y) = 0$$, $$u_S(x)$$, and $$u_N(x)=1$$, for all cases.

### Numerical Solution Approach.

First, we are going to derive and implement a time marching method for the index$$-2$$ DAE system represented above. The time marching scheme will be implicit in the viscous term, and at least second-order accurate for the variables $q=\left( u,v\right)^T$. Moreover, we are going to develop an algorithm such that desired properties as a discrete conservation principle for mass, mometnum, and kinetic energy are preserved. Also, mimetic properties such that the scheme follow laws of differential calculus, and commutivity of operators will be enforced, see [[1]](#1) for more details.

The continuity equation, this is, $$\nabla \cdot \textbf{u} = 0$$, is going to be discretized in the following manner

$$
\nabla \cdot \textbf{u} = \frac{\partial u}{\partial x} + \frac{\partial v}{\partial y} = 0 
$$

$$
\quad \quad \;\, \approx \frac{u_{i+1,j} - u_{i,j}}{h_x} - \frac{v_{i,j+1} - v_{i,j}}{h_y} = 0 
$$

$$
\quad \quad \;\, \approx \textbf{D} \, \textbf{u} = \left[D_x, D_y \right] \textbf{u} = 0 
$$


The gradient of the pressure, i.e., $$\nabla p$$, is going to be discretize as 

$$
  \nabla p = \left( \frac{\partial p}{\partial x}, \frac{\partial p}{\partial y} \right)^\top
$$

$$
\quad \;\, \approx \left( \frac{p_{i,j} - p_{i-1,j}}{h_x}, \frac{p_{i,j} - p_{i,j-1}}{h_y} \right)^\top
$$

$$
\quad \;\,  \approx \textbf{D} \, p = \left[D_x, D_y \right] 
$$

The vorticity field. i.e., $$\nabla \times \textbf{u}$$, is going to be discretize as 

$$
  \nabla \times \textbf{u} = \frac{\partial v}{\partial x} - \frac{\partial u}{\partial y}
$$

$$
\quad \quad \;\, \approx \frac{v_{i,j} - v_{i-1,j}}{h_x} - \frac{u_{i,j} - u_{i,j-1}}{h_y} 
$$

$$
\quad \quad  \;\, \approx \textbf{C} \, \textbf{u} = \left[C_y, C_x \right] \textbf{u}
$$

Equipped with last three equations, we can attempt to compute $$\nabla \cdot \left( \nabla \times \phi \right)$$ in its descrete version and check that indeed this quantity is zero. In other words, the discrete operator mimics the identity $$\nabla \cdot \left( \nabla \times \phi \right) = 0$$. 

The Laplacian of a vector fiels, i.e., $$\nabla^2 \textbf{u}$$, is going to be discretize as

$$
  \nabla^2 \textbf{u} = \nabla \left( \nabla \cdot \textbf{u} \right) - \nabla \times \left( \nabla \times \textbf{u} \right)   
$$

$$
\quad \quad \, \approx \textbf{L} \, \textbf{u} = -\textbf{D} \, \textbf{G} \,  \textbf{u} - \textbf{C} \, \textbf{C}^T \, \textbf{u}   
$$


In order to employ the discrete stream-function, let define $$q = \textbf{C} \, s$$, then, discrete conservation of mass is automatically satisfied since $$\textbf{D} \, q = \textbf{D} \, \textbf{C} \, s = 0$$. Futhermore, we can write the vector field $$q$$ using the Helmholtz decomposition, this is basically $$q = \nabla \times \psi - \nabla \phi$$, then the operators that mimics the Helmholtz decomposition is $$q = \textbf{C} \, s - \textbf{D}^T \, f$$. Note that taking the divergence of the previous equation, we get

$$    
\textbf{D} \, q = \textbf{D} \, \textbf{C} \, s - \textbf{D} \, \textbf{D}^T \, f 
$$

$$
\quad \; \; \, = -\textbf{D} \, \textbf{D}^T \, f
$$

$$
\quad \; \; \,  = 0   
$$

Now, to take into account the boundary conditions, we introduce a layer of ``ghost`` cells, and set $$\textbf{u}$$ velocities in them to match no-slip, this basically means that for the upper-layer, we have

$$  
     \frac{1}{2} \left( u_{i,n} + u_{i,n+1} \right)  = 0
$$

And for the lower-layer

$$     
\frac{1}{2} \left( u_{i,0} + u_{i,1} \right)  = 0
$$

This will introduce some inhomogeneous term from no-slip boundary conditions, which can be written as $$b = \left[ b_u, b_v\right]^\top$$. Finally the advection term, after some algebra, can be written as

$$      
\left( u \otimes u \right)_{i,j} = \frac{1}{4} \left( u_{i,j} + u_{i,j-1} \right)^2 
$$

$$
\left( v \otimes v \right)_{i,j} = \frac{1}{4} \left( v_{i-1,j} + v_{i,j} \right)^2  
$$

$$
\left( u \otimes v \right)_{i,j} = \frac{1}{4} \left( u_{i,j} + u_{i,j-1} \right) \left( v_{i-1,j} + v_{i,j} \right)  
$$

Which will allow us to discretize the non-dimensionalize incompresible Navier-Stokes equations. Now we want to solve

$$   
\frac{\partial u}{\partial t} + \frac{\partial u^2}{\partial x} + \frac{\partial u \, v}{\partial y} = -\frac{\partial p}{\partial x} + \frac{1}{Re} \nabla^2 \, u 
$$

$$
\frac{\partial v}{\partial t} + \frac{\partial u \, v}{\partial x} + \frac{\partial v^2}{\partial y} = -\frac{\partial p}{\partial y} + \frac{1}{Re} \nabla^2 \, v  
$$

$$
\frac{\partial u}{\partial x} + \frac{\partial v}{\partial y} = 0  
$$

Which can be written in its discrete version as

$$     
    \frac{\partial q}{\partial t} = -\textbf{N}(q) - \textbf{G}\,p + \frac{1}{Re} \textbf{L} \, q + \frac{1}{Re} b 
$$

$$
    \textbf{D} \, q = 0
$$

### The Employed Numerical Scheme.

The previous equation is known as the semi-discrete incompressible Navier-Stokes equations, and correspond to a DAE-index 2 system. In order to solve it,  we will employ the null-space approach. Basically we will take advantage of the fact that  $$\textbf{D} \, q = \textbf{D} \, \textbf{C} \, s = 0$$, and therefore we will multiply this equation by $$\textbf{C}^T$$ and get

$$     
\textbf{C}^T \textbf{C} \frac{\partial s}{\partial t} = -\textbf{C}^T \, \textbf{N}(\textbf{C} \,s) + \frac{1}{Re} \textbf{C}^T \, \textbf{L} \, \textbf{C} \, s + \frac{1}{Re} \textbf{C} \,b \,.
$$

Note that the later equation eliminates the pressure term from momentum equation, and also mimetic the vorticity equation, now we are in position to employ an split Trapezoidal scheme for the viscous term, and a AB2 scheme for the advection term. In tis regard, we end up with the following system

$$
    \textbf{C}^T \left( \textbf{I} - \frac{\Delta t}{2 \, Re} \textbf{L} \right) \textbf{C} \;  \Delta s^n = \Delta t \; r^n \,,   
$$

where

$$
r^n = -\frac{3}{2} \textbf{C}^T \, \textbf{N}(\textbf{C} \,s^n) + \frac{1}{2} \textbf{C}^T \, \textbf{N}(\textbf{C} \,s^{n-1}) + \frac{1}{Re} \textbf{C}^T \, \textbf{L} \, \textbf{C} \, s^n + \frac{1}{Re} \textbf{C} \,b \,,
$$

and then, the stream-function is updated as

$$
    s^{n+1} = s^n + \Delta s^n
$$

The linear system will be solved using the operator $$x = A \backslash b$$ in MATLAB. This operator uses different options depending on the structure of the given matrix. For instance, the CHOLMOD option is used for sparse symmetric positive definite matrices. If the matrix is sparse, symmetric, and indefinite, then it uses MA49 developed by Iain Duff. For general sparse square matrices it uses UMFPACK. For sparse rectangular matrices it uses SuiteSparseQR. Finally, for sparse matrices that are lower or upper triangular, or can be permuted into such, it uses a forward/back solver that exploits these properties. 

In our particular case the matrix $$\textbf{A}$$ is given as 

$$
\textbf{A} = \textbf{C}^T \left( \textbf{I} - \frac{\Delta t}{2 \, Re} \textbf{L} \right) \textbf{C} \,.
$$

In order to know what option MATLAB is doing inside $$A \backslash b$$ we can use the option **spparms ('spumoni',3)**, and to turn it off we use **spparms ('spumoni',0)**. In this regard, we notice that MATLAB uses ***CHOLMOD's triangular solver***. This option makes perfect sence, since the structure of the matrix $$\textbf{A}$$ is not only sparse, but also symmetric and positive-definite. 

Now to see the efficiency of the CHOLMOD method, we will test it employing a mesh for the 2D lid-driven cavity problem with reasonable amount of grids points. In this analysis we have employed a grid of $$1000 \times 1000$$ in a computer with the following characteristics:

![image-title-here](/assets/img/Project4/Stats.png){:class="img-responsive"}{:height="" width="650px"}


The obtained CPU's time are shown in Table 1. These values allow us to conclude that Matlab does use parallelization while it solves the linear systems, however there is an important overhead in transfering data back and forth which does not allow a noticeable speedup. 

![image-title-here](/assets/img/Project4/TableI.png){:class="img-responsive"}{:height="" width="500px"}

### The 2D Lid--Driven Cavity Problem.

In this section, we will employ the code to solve the 2D lid--driven cavity flow with $$L/H = 1$$ and $$Re=100$$, since it is known that a steady state solution can be achieved. We have considered $$t_{sim} = 1000$$ to guarantee stationarity, $$n_x = n_y = 51$$ points along $$x$$ and $$y$$ directions, and $$\Delta t = 0.1$$. 

Figure 1 represents the contours for the the stream--lines once the flow has reached the steady-state. We can see that they match reference [[2]](#2). 

![image-title-here](/assets/img/Project4/Figure03.png){:class="img-responsive"}{:height="" width="350px"}

*Figure 1: Steady--stare contour streamline solution for the lid-driven cavity flow.*

Figure 2 compares the obtained results for the velocity field $$u(0.5,y)$$, and $$v(x,0.5)$$ with the one presented in the literature for the case $$Re = 100$$. 

![image-title-here](/assets/img/Project4/Figure01.png){:class="img-responsive"}{:height="" width="325px"}
![image-title-here](/assets/img/Project4/Figure02.png){:class="img-responsive"}{:height="" width="325px"}

*Figure 2: (a) Velocity field: $$u(x=0.5,y)$$. (b) Velocity field: $$v(x,y=0.5)$$.*

Both Figures 1 and  2 agree very well with the one provided in reference [[2]](#2). This essentialy means that our solver is working perfectly. 

Since, we have made sure our solver is working properly, we are going to show the evolution of the lid-driven cavity flow for different Reynold's numbers. 

![image-title-here](/assets/img/Project4/Re_1.png){:class="img-responsive"}{:height="" width="700px"}
*Figure 3: Time evolution of the lid-driven cavity flow for $$R_e = 1$$.*

![image-title-here](/assets/img/Project4/Re_100.png){:class="img-responsive"}{:height="" width="700px"}
*Figure 4: Time evolution of the lid-driven cavity flow for $$R_e = 100$$.*

![image-title-here](/assets/img/Project4/Re_1000.png){:class="img-responsive"}{:height="" width="700px"}
*Figure 5: Time evolution of the lid-driven cavity flow for $$R_e = 1000$$.*

![image-title-here](/assets/img/Project4/Re_10000.png){:class="img-responsive"}{:height="" width="700px"}
*Figure 6: Time evolution of the lid-driven cavity flow for $$R_e = 10000$$.*

The presented set of simulations for different Reynold's number allows us to conclude that the stationary state depend strongly on this parameter. The smaller the Raynold number is the most likely that the flow will reached stationarity in a smaller period. For the analyzed cases, we see that stationarity is reached roughly when the simulation time is larger than the Reynolds number -- not showed in the figures, but easily reproducible using the attached code in Matlab. 

Now, in order to demonstrate second-order spatio-temporal convergence, we will compare the solution of coarser meshes with a fine-mesh solution, i.e. $$n_x = n_y = 500$$. The error made for the stream-functions -- aka $$s(x,y)$$ -- was computed for differents values of $$n_x = n_y$$. We have chosen the $$L_2(\Omega)$$-norm as measurement of the error over the domain, which is defined in the following manner 

$$
  \| error \|_{L_2(\Omega)} = \sqrt{ \frac{1}{n_x} \frac{1}{n_y} \sum_{i=1}^{n_x}  \sum_{j=1}^{n_y} \| s(x_i,y_j) - \tilde{s}(x_i,y_j) \|^2} 
$$

For this analysis we have chosen $$n_x = n_y = 10$$, $$n_x = n_y = 20$$, $$n_x = n_y = 25$$, $$n_x = n_y = 50$$, and $$n_x = n_y = 100$$.  Figure 7 shows that in fact, we get approximate second order spatio-temporal convergence. 

![image-title-here](/assets/img/Project4/Figure04.png){:class="img-responsive"}{:height="" width="400px"}

*Figure 7: Second-order convergence of the Trapezoidal/AB2 scheme.*


### Some More Advanced Analysis.

We will consider other alternatives to solve the large system of equations that arise in our time march of the DAE system. In order to do this, we will fix a grid size, and we will compute the optimal time step to preserve accuracy. In this case we have chosen $$n_x = n_y = 51$$, $$t_{sim} = 1000$$, and $$R_e = 100$$. 

![image-title-here](/assets/img/Project4/TableII.png){:class="img-responsive"}{:height="" width="500px"}

Table 2 shows that the optimal $$\Delta t$$ in terms of speed/efficiency for this case is $$\Delta t = 0.5$$. Obviously, the larger the time step, the faster the code will run because fewer iteration are needed to reach the simulated time. 

Next, to see if the code can be accelerated, we could attempt to employ another alternatives to solve the linear system,  since the performance of this code only depends on the speed of solving the linear system $$x = A \backslash b$$. We will explore several iterative alternatives to deal with this task. We are going to consider a mesh-grid with $$n_x = n_y = 10, 100, 1000$$ points, and a tolerance of $$10^{-6}$$ for convergence. The methods explored are the reconditioned conjugate gradients method (**pcg**), biconjugate gradients method (**bicg**), generalized minimum residual method (**gmres**), and minimum residual method (**minres**). 

The results of this extensive analysis are shown in Table 3.

![image-title-here](/assets/img/Project4/TableIII.png){:class="img-responsive"}{:height="" width="500px"}

 
Table 3 shows that the faster routine is **mldivide**, which as we mentioned before, it  employs the ***CHOLMOD's triangular solver*** which is the best option due to the structure of the provided $$\textbf{A}$$ matrix. I have also tried to solve the system using a modified version of the $$LU$$ factorization, provided in the appendix of this document, called **luSolve(A,b)** which requires a computing time of $$31.28 \; [s]$$ for $$n_x = n_y = 1000$$ points, which is not bad compared with the super--optimized **mldivide** routine implemented in Matlab. 

Finally, we will analyze the time marching method and estimate a time step restrictions by choosing different values of of $$R_e$$ and $$\Delta x$$. For this analysis we will fix the Reynold's number to be $$R_e = 100$$, so that our space discretization, aka $$\Delta x$$, will move from $$0.1$$ to $$0.01$$. This discretization will allow us not only to change the time step, aka $$\Delta t$$, from $$0.10$$ to $$10.0$$ but also to see inestabilities in the scheme.  

The reults of this exhaustive analysis are shown in Figure 8. Here the solids red circles represents that the scheme was able to run the case for a long time, here $$t_{sim} = 1000$$, and the empty circle represents overflow of the method. The dashed blue line represents an approximate limit for this case. 

![image-title-here](/assets/img/Project4/Figure05.png){:class="img-responsive"}{:height="" width="400px"}

*Figure 8: Time step restrictions of the Trapezoidal/AB2 scheme for $$R_e = 100$$.*

The same analysis can be done for several different Reynolds numbers, but for the sake of brevity, I have omited them. The behavior is cualitatively the same as the one presented in Figure 8.

### References
<a id="1">[1]</a> 
Colonius, Tim.
Lecture notes -- Computational Fluid Dynamics
California Institute of Technology, Fall 2016.

<a id="1">[2]</a> 
U. Ghia, K.N. Ghia, and C.T. Shin. 
High-Re solutions for incompressible flow using the Navier-Stokes equations and a multigrid method
Journal of Computational Physics, 48:387--411, 1982.

<a id="1">[3]</a> 
Ferziger, J.H, and Peri\'c, M.
Computational Methods for Fluid Dynamics
3rd Edition, Springer.