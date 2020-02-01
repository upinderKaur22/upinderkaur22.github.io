---
layout: page
title: Project III
description: A Matlab Galerkin spectral Fourier Method for Navier Stokes.
img: /assets/img/Project3/Background.png
---


The main objective of this project is to implement and use an spectral code to solve the two-dimensional, incompressible 
Navier-Stokes equations in a periodic domain. We have chosen the Fourier collocation method to deal with this task. Basically, 
we use the vorticity stream formulation for the implementation and then we compute the velocity field from the stream function. 
The idea is to use our implementation to get a better understanding of the dependency on initial condition in the Navier-Stokes 
equations as well as to study the convergence and stability of the employed numerical method. 


### The incompressible Navier--Stokes equation.
The dimensionless incompressible Navier--Stokes equation in its traditional form for solving the velocity field is given as 
follows

$$
    \partial_t \textbf{u} + \textbf{u} \cdot \nabla \textbf{u} = \frac{1}{Re} \, \nabla^2 \textbf{u} \,,
$$

$$  
    \nabla \cdot \textbf{u} = 0 \,,
$$

where $$Re$$ represents the Reynold's number, and $$\textbf{u}$$ the velocity field in two-dimensions which is given by $$\textbf{u}(x,y)=\left[ u(x,y), v(x,y) \right]^T$$. 

In order to derive the vorticity stream function for the Navier-Stokes equation in two dimensions, we apply the $$curl$$ operator. The following procedure is a common way of deriving the vorticity equation. The vorticity is thus defined as 

$$
  w = \nabla \times \textbf{u}  \,.
$$

We also know that the following vectorial identity holds

$$
    \frac{1}{2} \nabla \left( \textbf{u} \cdot \textbf{u} \right) = \left(\textbf{u} \cdot \nabla \right) \textbf{u} + \textbf{u} \times \left( \nabla \times \textbf{u} \right) \,,
$$

hence, for an scalar function $$\phi(x,y)$$  the following relation holds

$$
 \nabla \times \nabla \phi = 0 \,. 
$$

Therefore re-writing the latter identity in terms of $$\textbf{u}$$ and $$w$$ we get

$$
   \nabla \times \left( \textbf{u} \times w \right) = \left( w \cdot \nabla \right) \textbf{u} - \left( \textbf{u} \cdot \nabla \right)w + \textbf{u} \nabla \cdot w - w\nabla \cdot \textbf{u} \,,
$$

where the last two terms vanish since $$\nabla \cdot w = 0$$ and $$\nabla \cdot \textbf{u} = 0$$. Employing the above identities, we can transform the Navier-Stokes equation into

$$
    \partial_t \textbf{u} +  \frac{1}{2} \nabla \left( \textbf{u} \cdot \textbf{u} \right) - \textbf{u} \times \left( \nabla \times \textbf{u} \right)  = \frac{1}{Re} \, \nabla^2 \textbf{u} \,.
$$

We can take the $$curl$$ on both sides of the equation. Using the identities previously mentioned, we get the ***vorticity equation***:

$$
  \partial_t w + \left( \textbf{u} \cdot \nabla \right) w - \left( w \cdot \nabla \right) \textbf{u} = \frac{1}{Re} \, \nabla^2 \textbf{u} \,.
$$

This gives us the two--dimensional vorticity equation

$$
  \partial_t w = - \frac{1}{Re} \, \nabla^2 w - \nabla \left( w \, \textbf{u} \right) 
$$

The above is the main equation we want to consider. This equation is a nonlinear advection diffusion equation. Once we can successfully solve for vorticity we solve for stream function $$\psi(x,y)$$ which is defined as follows 

$$
  w = -\nabla \psi
$$

### The Warm-up Steps to Get Started.

This section's purpose is to make the end job easier by setting-up several simple tasks. In order to achieve this purpose, we are going to emply Matlab as our programming environment; therefore, we do not need to install any Fast Fourier Transform (FFT)  package, since the built-in function ***fft()*** carries out this task. 

In order to understand how the ***fft()*** function works in Matlab, let us first define the following analytical function  

$$
  g(x) = \exp\left(-a x^2 \right) 
$$

Now, let us compute the continuous Fourier transform over the real interval defined when $$x \in \left[-\infty,\infty \right]$$ for $$g(x)$$ as follows  

$$
    \tilde{g}(k) = \int_{-\infty}^{\infty} \exp\left(-a x^2 \right) \exp\left(-2 \pi i \,  k x\right)\, dx = \frac{1}{2\sqrt{\pi a}}\exp\left(-\frac{k^2}{4 a} \right)
$$

If we consider the values $$a = 0.5$$, $$N = 64$$, and a windows $$x \in \left[ -\pi,\pi \right]$$ for $$\tilde{g}(k)$$, we can plot the discrepancy between continuous Fourier transform, and the discrete fourier transform. Figure 1 shows in fact this results.  

![image-title-here](/assets/img/Project3/WarmUp01.png){:class="img-responsive"}{:height="" width="500px"}

*Figure 1: (a) The analytical function $$g(x)$$, (b) The Fourier coefficients comparison.*

Now we are in position of changing the values of $$N$$ and $$a$$, and by comparison of the coefficients for the two functions and their analytical counterpart, we will isolate and evaluate three potential errors: truncation of the Fourier coefficients, sampling/aliasing, and non-periodicity errors.

To meassure the error, we have chosen the $$L_2(\Omega)$$-norm  defined as

$$
  \| g(k) \|_{L_2(\Omega)} = \sqrt{ \frac{1}{N} \sum_{i=1}^{N} \| g(k_i) - \tilde{g}(k_i) \|^2} 
$$

Table 1 summarizes these findings.

![image-title-here](/assets/img/Project3/TableI.png){:class="img-responsive"}{:height="" width="550px"}

Table 1 shows that the ***truncation error*** comes from the fact of employing an small number $$N$$. That is why we see for a given $$a$$, the error decreases if the number of ampling points $$N$$ increases.

On the other hand, since sampling is the process of discretizing the domain of a continuous function to produce a discrete function, it is not surprising that if the sampling rate is too low, information is lost and the continuous funtion is not uniquely determined by the samples. This kind of phenomenon is called ***aliasing error*** and that is why we see that the error decreases faster when the value of $$a$$ is much larger because we need to sample more. 

Now, to make things a bit more interesting, let us consider the following analytic two-dimensional function  

$$
  g(x,y) = \exp\left(-a \left( x^2 +y^2\right) \right) 
$$

The Fourier coefficients of $$g(x,y)$$ are computed in the following manner

$$
    \tilde{g}(j,k) = \int_{-\pi}^{\pi} \int_{-\pi}^{\pi} \exp\left(-a \left( x^2 + y^2 \right) \right) \exp\left(-2 \pi i \left( j x + ky \right) \right)  \, dx \,dy 
$$

$$
    \quad \quad \quad = \frac{1}{4 \pi a}\exp\left(-\frac{\pi^2}{4a}\left( j^2 + k^2\right) \right)
$$

For numerical visualization, let us compare the coefficients obtained using the analytical expression and the one obtained using ***fft2()*** in Matlab. For this purpose, we considered the values $$a = 2.5$$, $$N = 128$$ over the domain $$(x,y) \in \left[-\pi,\pi\right]^2$$.

![image-title-here](/assets/img/Project3/WarmUp03.png){:class="img-responsive"}{:height="" width="600px"}

*Figure 2: Fourier coefficients comparison.*

Now we could also compute the error made in the coefficients for differents values of $$N$$ and $$a$$. Once again, we have chosen the $$L_2(\Omega)$$-norm as measurement of the error over the domain. 

$$
  \| g(j,k) \|_{L_2(\Omega)} = \sqrt{ \frac{1}{N} \frac{1}{N} \sum_{i=1}^{N}  \sum_{l=1}^{N} \| h(j_i,k_l) - \tilde{h}(j_i,k_l) \|^2} 
$$

Table 2 summarizes these findings.

![image-title-here](/assets/img/Project3/TableII.png){:class="img-responsive"}{:height="" width="550px"}


Table 2 again shows that the ***truncation error*** is decreased when a larger number of sampling points $$N$$ is employed. Morever, the ***aliasing error*** also decreases faster when the value of $$a$$ is larger.

At this point, we are in possition to write our first Fourier-spectral solver for the (vorticity-stream function) Poisson equation:

$$
  \frac{\partial^2 \psi}{\partial x^2} + \frac{\partial^2 \psi}{\partial y^2} = -w  \,,
$$

here $$w(x,y)$$ was provided in the appendix of this project. The analytical Taylor vortex function for this particular case is

$$
  w(x,y) = \left( 1 - \frac{x^2 + y^2}{2}\right) \exp\left( \frac{1}{2} \left( 1 - x^2 - y^2\right) \right) \,. 
$$

The analytic solution for $$\psi(x,y)$$ can be obtained by means of integration  of the provided expression

$$
  \psi(x,y) = -\exp\left( \frac{1}{2} \left( 1 - x^2 - y^2\right) \right) \,.
$$
 
The domain is considered periodic over the domain $$(x,y) \in \left[-2\pi,2\pi\right]^2$$. Now, we assume that the solution of $$\psi(x,y)$$ can be written as follows

$$
   \psi(x,y) = \sum_{j=-M/2}^{M/2-1} \sum_{k=-M/2}^{M/2-1} \tilde{\psi}_{jk} \exp \left( 2 \pi i \left( jx + ky \right) \right) \,.
$$

Replacing equation previous equation into the Poisson's equation, we obtain

$$
  \left( \epsilon_x^2 + \epsilon_y^2 \right) \tilde{\psi}_{jk} = \tilde{w}_{jk} \,,
$$

here $$\epsilon_x = 2 \pi i/M$$, and $$\epsilon_y = 2 \pi i/M$$. Solving for the coefficients $$\tilde{\psi}_{jk}$$, we are in position to get back to the physical domain using the ***ifft2()*** function in Matlab. Figure 3 shows the results obtained considering $$M=256$$. 

![image-title-here](/assets/img/Project3/WarmUp04.png){:class="img-responsive"}{:height="" width="650px"}

*Figure 3: Fourier-spectral solver for the Poisson equation.*

To measure quantitatively the error made by truncation and alaising, we have employed the $$L_2$$-norm. Table 3 summarizes these findings. 

![image-title-here](/assets/img/Project3/TableIII.png){:class="img-responsive"}{:height="" width="550px"}


Table 3 shows that the error decreases when a larger number $$N$$ is employed, this is basically because the approximation becomes more accurate since the error made by truncation is smaller, and the alaising component was taken care by increasing the size of the domain. 

The last task of this warm-up will be to write a program to evaluate the discrete Fourier coefficients of products of functions. For the sake of simplicity, we will use $$g(x,y)=\exp\left(-q\left(x^2+y^2\right)\right)$$ multiplied by itself, since we can evaluate the exact fourier coefficients of this new function $$h(x,y)$$ analytically. 

The Fourier coefficients of $$h(x,y) = g^2(x,y)$$ are computed as follows

$$
  \tilde{h}(j,k) = \frac{1}{8 \pi a}\exp\left(-\frac{\pi^2}{8a}\left( j^2 + k^2\right) \right)
$$

For numerical visualization, let us compare the coefficients obtained using the analytical expression $$\tilde{h}(j,k)$$ and the one obtained using the di-aliased method. For this purpose, we considered the values $$a = 1.0$$, $$N = 128$$ over the domain $$(x,y) \in \left[-\pi,\pi\right]^2$$. 

![image-title-here](/assets/img/Project3/WarmUp05.png){:class="img-responsive"}{:height="" width="600px"}

*Figure 4: Di--Aliased Fourier coefficients for $$\tilde{h}(j,k)$$.*

Now we could also compute the error made in the coefficients for differents values of $$N$$ and $$a$$. Once again, we have chosen the $$L_2(\Omega)$$-norm as measurement of the error over the domain. This result is summarized in Table 4. 

![image-title-here](/assets/img/Project3/TableIV.png){:class="img-responsive"}{:height="" width="550px"}


### The Employed Numerical Scheme.
We use Fourier-spectral method for solving the incompresible Navier--Stokes equations 

$$\partial_t w = - \frac{1}{Re} \, \nabla^2 w - \nabla \left( w \, \textbf{u} \right) \,,$$ 

so if we assume that the solution of this equation can be writen as

$$
   \psi(x,y,t) = \sum_{j=-M/2}^{M/2-1} \sum_{k=-M/2}^{M/2-1} \tilde{\psi}_{jk}(t) \exp \left( 2 \pi i \left( jx + ky \right) \right) \,,
$$
 
and, if we replace it into the the vorticity equation, we get

$$
  \frac{\partial}{\partial t} \tilde{w}_{jk} = - \frac{1}{Re} \left( \epsilon_x^2 + \epsilon_y^2 \right) \tilde{w}_{jk} - \left( \epsilon_x \; \tilde{u} \cdot \tilde{w} \right)_{jk}  - \left( \epsilon_y \; \tilde{v} \cdot \tilde{w} \right)_{jk} \,,
$$

where

$$
    \epsilon_x = \frac{2 \pi \, i}{M} \,,
$$

$$
    \epsilon_y = \frac{2 \pi \, i}{N} \,,
$$

$$
    \tilde{\psi}_{jk} =  \frac{ \tilde{w}_{jk}}{\alpha^2 \, j^2 + \beta^2 \, k^2} \,,
$$

$$
    \tilde{u}_{jk}    =  \frac{ i \, \beta  k \tilde{w}_{jk}}{\alpha^2 \, j^2 + \beta^2 \, k^2} \,,
$$

$$
    \tilde{v}_{jk}    = -\frac{ i \, \alpha j \tilde{w}_{jk}}{\alpha^2 \, j^2 + \beta^2 \, k^2} \,,
$$

The right hand side of $$\frac{\partial}{\partial t} \tilde{w}_{jk}$$ can be solved using discrete Fourier transform on the grid points as it was done in the warm-up section. We have used the ***fft2()*** algorithm implemented in Matlab to solve the right hand side. For time evolution we have implemented the Runge--Kutta fourth--order implicit method. 

We also have to take care of the aliasing problem by throwing out the frequencies in the convective term $$\nabla \left( w \, \textbf{u} \right)$$ employing the method specified in the warm--up. This procedure is implemented in the function ***DiAliasedFunction()*** for more details.

### Algorithm and Convergence.

In order to ***empirically determine the stability bounds*** of the time marching scheme, we will initialize the Fourier coefficients to random complex numbers. We will also run several cases with different values of time step $$\Delta t$$, grid spacing $$h$$, and ***Reynold*** numbers, and then we will compare them with the stability limits for the $$RK4$$ integration sheme provided bellow:

$$
  \Delta t_{max} = \min \left( \frac{2.8 \, h}{\pi \, V_{max}} , \frac{2.8 \, Re \, h^2}{\pi^2} \right) \,.
$$
 
The physical domain was taken as $$(x,y) \in \left[-\pi,\pi\right]^2$$. We have considered the discretization in the Reynold numbers to be $$0.01$$, and $$100$$. For the grid spacing, the number of sampling points $$N$$ was taken to be uniform in both directions and equal to $$4$$, $$8$$, $$32$$, $$64$$, $$128$$, $$256$$, $$512$$, and $$1024$$. Finally, for the time step $$\Delta t$$, we employed the values $$1.0$$, $$0.1$$, $$0.01$$, $$0.001$$, $$0.0001$$.  

The reults of this exhaustive analysis are shown in Figure 5 and 6. Here the solids black circles represents that the scheme was able to run the case for a long time, here $$t_{sim} = 12.5$$, and the empty circle represents overflow of the method. 

![image-title-here](/assets/img/Project3/Algorithm01a.png){:class="img-responsive"}{:height="" width="400px"}

*Figure 5: Empirical determination of $$\Delta t$$ for $$Re = 0.01$$.*

![image-title-here](/assets/img/Project3/Algorithm01b.png){:class="img-responsive"}{:height="" width="400px"}

*Figure 6: Empirical determination of $$\Delta t$$ for $$Re = 100.0$$.*

We see clearly, that the bound specified for $$\Delta t_{max}$$ agrees very well the convergence criterion for selecting the time step. 

Next, to ***demonstrate convergence to an exact solution***, we will compute the solution for the Taylor vortex for relatively short times, so that our solution should be very close to the exact solution.

First, we will hold the time-step constant, here $$t = 0.001$$ but small enough so that the error is dominated by the spatial discretization. The spatial domain was taken to be $$(x,y) = \left[ -2 \pi,2 \pi\right]^2$$, and the simulation time $$t_{sim} = 1.5$$. The spectral convergence is shown in Figure 7. We see clearly that while the amount of grid ponts $$N$$ is increased, the error accordingly decreases very fast as expected. 

![image-title-here](/assets/img/Project3/Algorithm02a.png){:class="img-responsive"}{:height="" width="400px"}

*Figure 7: The spectral convergence Fourier-spectral scheme.*

Second to show the fourth-order convergence of the Runge-Kutta scheme, we will decrease the time step in proportion to the grid spacing. Since the spectral discretization error decreases rapidly for sufficiently large meshes, we expect to be able to see this behavior with some difficulties since the machine presition is reached very fast due to the spectral behavior of the method.  

![image-title-here](/assets/img/Project3/Algorithm02b.png){:class="img-responsive"}{:height="" width="400px"}

*Figure 8: Fourth-order convergence of the Runge-Kutta scheme.*

In Figure 8, we have considered $$N$$ to be $$128$$, $$136$$, $$144$$, $$192$$, and $$256$$, the time steps $$\Delta t$$ were taken as $$0.15$$, $$0.065$$, $$0.010$$, $$0.0055$$, and $$0.0015$$, the error was meassured employing the $$L_2$$-norm as usual. We see in Figure 8 that we almost get the fourth-order convergence $$\sim 3.7$$, the variation in the slope might be related to the lack of presition in the last values of $$N$$.

Finally, to demonstrate the ***spatial convergence to a numerical solution*** on a very fine grid, we compute a reference solution which is a ultra-fine mesh in time and space, and then we compare the solution on coarser meshes. In this analysis we have employed $$N = 2000$$, and $$\Delta t = 0.0001$$. We expect that the differences between the coarser mesh solutions is sufficiently large that the difference between the fine mesh and the exact solutions are insignificant.

Figure 9 shows in a log-linear scale the decrease in the error as $$N$$ increases. It is important to point out that the coefficient obtained in this regard ($$0.22$$) is less than the one specified in the project, however this analysis still confirms the spectral nature of this method. 

![image-title-here](/assets/img/Project3/Algorithm03.png){:class="img-responsive"}{:height="" width="400px"}

*Figure 9: Spatial convergence to a numerical solution.*

We have employed in this analysis $$N$$ to be $$50$$, $$100$$, and $$200$$. The spatial domain was taken to be $$(x,y) = \left[ -2 \pi,2 \pi\right]^2$$, and the simulation time was fixed at  $$t_{sim} = 1.0$$.


### Some Interesting Selected Examples.

Let us start ***Dynamics of a co-rotating vortex pair***. We will initialize two Taylor vortices with core size $$c = \pi/10$$, equal maximum velocity, and separated by a distance $$d = \pi/6$$. This setting will allow the two vortex to form a single (wobbly) vortex. Figure 10 shows the evolution of this phenomenon

![image-title-here](/assets/img/Project3/Fun01.png){:class="img-responsive"}{:height="" width="700px"}
*Figure 10: Dynamics of a co-rotating vortex pair.*

In order to generate Figure 10 we have considered the domain to be $$(x,y) \in \left[-\pi,\pi\right]^2$$, the number os sampling points $$N = 256$$, the maximum time step $$\Delta t = 0.005$$, and the Reynold's number $$Re = 350$$.

Next, we will check the stability of a ***Periodic array of vortices***. In order to do this, we will initialize the domain with a doubly-periodic array of alternating sign vortices. We expect that if there are no external disturbances that break the symmetry, the vortices should be stationary and simply diffuse in time. This is in fact was is shown in Figure 11

![image-title-here](/assets/img/Project3/Fun02.png){:class="img-responsive"}{:height="" width="700px"}
*Figure 11: Evolution of a periodic array of vortices*

In order to generate Figure 11 we have considered the domain to be $$(x,y) \in \left[-\pi,\pi\right]^2$$, the number os sampling points $$N = 256$$, the maximum time step $$\Delta t = 0.005$$, the Reynold's number $$Re = 350$$.

Next, we consider the ***Two-dimensional turbulence***. We will initialize our domain with Fourier coefficients of equal magnitude -- for this case we have employed the amplitud to be one -- but randomly chosen phases. We also excited only the portion of wavenumbers in the range of $$\mid \alpha \mid \leq M/4$$ , $$ \mid \beta \mid \leq N/4$$. Figure 12 shows the evolution of this case.

![image-title-here](/assets/img/Project3/Fun03.png){:class="img-responsive"}{:height="" width="700px"}
*Figure 12: Evolution of a two-dimensional turbulence.*

In order to generate Figure 12 we have considered the domain to be $$(x,y) \in \left[-\pi,\pi\right]^2$$, the number os sampling points $$N = 256$$, the maximum time step was taken as $$\Delta t = 0.005$$, the simulation time $$t_{sim} = 12.50$$, and the Reynold's number $$Re = 350$$.

Finally, we will study the ***Temporally evolving jet***. In order to accomplish this task, we will create two equal and opposite mixing layers separated by a distance $$a = \pi$$. We have also added some perturbations to observe the Kelvin-Helmholtz instability shown in Figure 13.

![image-title-here](/assets/img/Project3/Fun04.png){:class="img-responsive"}{:height="" width="700px"}
*Figure 13: Temporally evolving jet.*

In order to generate Figure 13 we have considered the domain to be $$(x,y) \in \left[-\pi,\pi\right]^2$$, the number os sampling points $$N = 256$$, the maximum time step $$\Delta t = 0.005$$, the Reynold's number $$Re = 350$$. We have also employed a little noise, which was added to the initial condition, as a uniform random variable of amplitude 0.01.

### References
<a id="1">[1]</a> 
Colonius, Tim.
Lecture notes -- Computational Fluid Dynamics
California Institute of Technology, Fall 2016.

<a id="1">[2]</a> 
Ferziger, J.H, and Peri\'c, M.
Computational Methods for Fluid Dynamics
3rd Edition, Springer.
