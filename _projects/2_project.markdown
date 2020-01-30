---
layout: page
title: A GPU implementation of the Shallow Water Equations.
description: How to speedup the Shallow Water Equations on Graphic processors.
img: /assets/img/2.jpg
---

This project is  basically a numerical implementation on GPU of the Shallow Water Equation taken form \cite{Kurganov_1}, and \cite{Liang}. The chosen method employs a very efficient finite volume algorithm for the numerical solution of the \emph{Shallow Water Equations}, 
here SWE, which has not only the characteristic to be robust, and accurate, but also it is implemented in non-structured triangular mesh.

In general, method that solves the SWE must fulfill both the well-balanced property (this means must not generate synthetic perturbations for lake at rest conditions), and must preserve the positivity preserving property for the water depth, this means the water column must be always positive \cite{Kurganov_1}. 

Because of the previous reasons, an improved first-order central-upwind was chosen to be implemented, see \cite{Bryson_1}, which in contrast to their predecessors, is able to preserve the lake at rest conditions and the positivity water level \cite{Bryson_2}, employing a variable time step to increase the convergence in time. Moreover, in order to increase the stability of the method, it is necessary to increase the resolution of the mesh in coastal areas. This is carried out by employing a non-structured mesh. To fulfill these equirements triangular elements were employed.

### Model Definition.

In this section, it will be defined the main variables that are going to be employed during this document. We will use normal variables <img src="https://render.githubusercontent.com/render/math?math=q"> as scalar, while in bold <img src="https://render.githubusercontent.com/render/math?math=\mathbf{q}"> as vectors, i.e., we will have <img src="https://render.githubusercontent.com/render/math?math=\mathbf{q} = (q_1, q_2, q_3)^\top"> as a vector of conserved variables. 

<div class="img_row">
    <img class="col three left" src="{{ site.baseurl }}/assets/img/Project2/Fig001.png" alt="" height="10" title="example image"/>
</div>
<div class="col three caption">
    Main (state) variables used to describe the shallow water equations (SWE).
</div>

where in the figure <img src="https://render.githubusercontent.com/render/math?math=w(x,y,t)"> Represents the surface elevation. It is measured from mean sea level to the water level, <img src="https://render.githubusercontent.com/render/math?math=h(x,y,t)"> Represents the water depth. It is measured from the bed elevation to the water surface,  <img src="https://render.githubusercontent.com/render/math?math=u(x,y,t)"> Represents the velocity field along x-direction, <img src="https://render.githubusercontent.com/render/math?math=v(x,y,t)">  Represents the velocity field along y-direction, and <img src="https://render.githubusercontent.com/render/math?math=B(x,y)"> Represents the bathymetry. It is measured from the mean sea level to the bottom floor. Note that the associated variables  <img src="https://render.githubusercontent.com/render/math?math=hu(x,y,t)"> and <img src="https://render.githubusercontent.com/render/math?math=hv(x,y,t)"> Represents the flux-discharge along x and y direction respectively.

### The Shallow Water Equations.

Assuming hydrostatic pressure condition, the SWE are obtained by integrating the Navier-Stokes equations over the water depth. In this regard, a system of bi-dimensional equations is obtained, in which the horizontal velocities represents an average of the velocity along the water column. Neglecting the kinematic and turbulent terms, the SWE can be written as: 

<img src="https://render.githubusercontent.com/render/math?math=h_t %2B (hu)_x %2B (hv)_y = 0" height="75px">

<img src="https://render.githubusercontent.com/render/math?math=(hu)_t %2B \left(hu^2 %2B \frac{1}{2} g h^2 \right)_x %2B (huv)_y = -g h \z_x - g n^2 \frac{(hu) \sqrt{(hu)^2 %2B (hv)^2}}{h^{7/3}}" height="75px">

<img src="https://render.githubusercontent.com/render/math?math=(hv)_t %2B (huv)_x  %2B \left( hv^2 %2B \frac{1}{2} g h^2 \right)_y = -g h \z_y - g n^2 \frac{(hv) \sqrt{(hu)^2 %2B (hv)^2}}{h^{7/3}}" height="75px">

The first equation represent the mass balance due to the change of water height of the water column in a certain point. This value should be the discharge that the water column should have. The second and third equations represent the moment balance, and are related to the change in the discharge with the weight of the eater column, the bathymetry source, and the bottom friction's force, etc. 

These set of equations can be written down on its conserved vector form as follows: 

<img src="https://render.githubusercontent.com/render/math?math=\frac{\partial \mathbf{q}}{\partial t} %2B \frac{\partial \mathbf{f}(\mathbf{q})}{\partial x} %2B \frac{\partial \mathbf{g}(\mathbf{q})}{\partial y} = \mathbf{S}(\mathbf{q}) %2B \mathbf{R}(\mathbf{q})" height="75px">

where the previous variables represents:

<img src="https://render.githubusercontent.com/render/math?math=\mathbf{q} = \left[ h,  hu,  hv \right]^\top">

<img src="https://render.githubusercontent.com/render/math?math=\mathbf{f}(\mathbf{q}) = \left[ hu, hu + \frac{1}{2}gh^2, huv \right]^\top">

<img src="https://render.githubusercontent.com/render/math?math=\mathbf{g}(\mathbf{q}) = \left[ hv, huv, hv + \frac{1}{2}gh^2 \right]^\top">

<img src="https://render.githubusercontent.com/render/math?math=\mathbf{S}(\mathbf{q}) = \left[ 0, -gh \frac{\partial z}{\partial x}, -gh \frac{\partial z}{\partial y} \right]^\top">

<img src="https://render.githubusercontent.com/render/math?math=\mathbf{R}(\mathbf{q}) = \left[ 0, g n^2 \frac{(hu) \sqrt{(hu)^2 + (hv)^2}}{h^{7/3}}, g n^2 \frac{(hv) \sqrt{(hu)^2 + (hv)^2}}{h^{7/3}} \right]^\top">

where $\mathbf{S}(\mathbf{q})$: represents the source term, $\mathbf{R}(\mathbf{q})$: represents the bottom friction term, and $n$: is the Manning's roughness coefficient.

### A Finite Volume Discretization.

In order to solve the SWE, the finite volume method is employed. First, the flux-field will be written in its vector form as: <img src="https://render.githubusercontent.com/render/math?math=\overrightarrow{\mathbf{F}}(\mathbf{q}) = (\mathbf{f},\mathbf{g})">. Then, equation (\ref{VecForm}) will take the form:

<img src="https://render.githubusercontent.com/render/math?math=\frac{\partial \mathbf{q}}{\partial t}} %2B \nabla \cdot \overrightarrow{\mathbf{F}}(\mathbf{q}) = \mathbf{S}(\mathbf{q}) %2B \mathbf{C}(\mathbf{q}) %2B \mathbf{R}(\mathbf{q})">

Integrating by part over a finite triangular control volume <img src="https://render.githubusercontent.com/render/math?math=\Omega">, we get:
 
\begin{align}
\iint_{\Omega} \dt{\q} \; d \Omega + \iint_{\Omega} \nabla \cdot \overrightarrow{\F}(\q) \; d \Omega= \iint_{\Omega} \left( \S(\q) + \C(\q) + \R(\q) \right) \; d \Omega 
\end{align} 
<img src="https://render.githubusercontent.com/render/math?math=...">

Applying the Stokes' theorem to a finite triangular element, we obtain: 

\begin{align}
\dt{\q} \cdot \Omega + \oint_{\Gamma} \F(\q) \cdot \overrightarrow{n} \; d \Gamma = \S(\q) \cdot \Omega + \C(\q) \cdot \Omega + \R(\q) \cdot \Omega
\end{align}
<img src="https://render.githubusercontent.com/render/math?math=...">

Where, $\Omega$: is the control volume, $\Gamma$: is the boundary of the control volume, $\overrightarrow{n}$: is the normal vector to the side, and $\F(\q)$: is the normal flux to the edge of each side of the control volume. 


Employing the Euler's method to discretize the temporal variable, we get: 

\begin{align}\label{Euler}
\q_j^{m+1} = \q_j^{m} - \frac{\Delta t}{\Delta \Omega_j} \sum_{k=1}^{3} \F_{jk}(\q^{m}) \cdot n_{jk} \cdot l_{jk} + \Delta t \cdot \S_j(\q^{m})
\end{align}
<img src="https://render.githubusercontent.com/render/math?math=...">

Where $n_{jk}$: is the normal vector to the $k^{th}$ mid-point side of the $j^{th}$ triangle, $\Delta t$: is the time step, $\Delta \Omega_j$: is the control volume for the $j^{th}$ triangle, $l_{jk}$: is the length of the $k^{th}$ side of the $j^{th}$ triangle, and $\q_j^{m}$: is the vector of conserved variables for the $j^{th}$ triangle at the $m^{th}$ time step. 

Terms such as the bottom friction and Coriolis' force can be incorporated employing a semi-implicit discretization:

\begin{align}
\C(\q) + \R(\q) \approx   H(\q^{m+1}) \cdot \q^{m+1}
\end{align}
<img src="https://render.githubusercontent.com/render/math?math=...">

In this regard, the friction and the Coriolis terms can be incorporated correcting the vector of conserved variables:

\begin{align}\label{eq:update}
\q_j^{m+1} = \frac{\q_j^{m+1}}{1 + \Delta t \cdot H(\q_j^{m+1})} 
\end{align}
<img src="https://render.githubusercontent.com/render/math?math=...">

### First--Order Semi--Discrete Central Upwind.

In this section, the \emph{First--order semi--discrete central--upwind} method will be employed to solve the discrete form of the Sallow Water Equations presented in equation(\ref{Euler}). In order to get a better understanding of the method, it is a smart idea to use as a reference the scheme depicted in figure (\ref{Fig002}): 

\begin{figure}[ht] 
  \centering
  \includegraphics[scale=0.250]{Fig002.png}  
  \caption{(a) Finite volume mesh, (b) typical control volume and its associated variables.}
  \label{Fig002}
\end{figure}

The method starts by defining a piece-wise constant reconstruction of the bathymetry. The mass center value of the elements will be taken as constant over the element. Therefore, a discontinuous values will be provided at each side of the element, this is $in$ and the $out$ interface as it is shown in figure (\ref{fig:Fig003}). In order to obtain the final states of the conserved variables, a single value of bed elevation suggested by \cite{Klein} should be defined: 

\begin{align}\label{eq:DefBatimetria}
 B_{jk} = \max \left( B^{in}_{jk},B^{out}_{jk} \right)
\end{align}

The \emph{First--Order Semi--Discrete Central--Upwind Method} begins by knowing the explicit functions of the conserved variables at the mass center of each triangle at the previous time step, i.e., $\q_j^m$. Then, a piece-wise constant reconstruction will be employed, this is, the conserved variable values at the mid-point of each side are the same as the mass center. 

\begin{align}\label{eq:Reconstruccion}
\q_{jk}^{in} = \q_j^{m} 
\end{align}

The water level surface reconstruction must be so that the water depth has to be always positive; therefore, the reconstructed water depth at the midpoint $h_{jk}$ should be re-defined as: 

\begin{align}\label{eq:Profundidad}
h^{in}_{jk} = \max \left(Tol, w^{in}_{jk} - B_{jk} \right)
\end{align}

It is trivial to prove that the reconstruction process defined above do not affect the well-balanced property of the numerical scheme when only wet-bed applications are simulated. However, for dry-bed applications, the well-balanced property is violated, see figure(\ref{fig:Fig003}).

\begin{figure}[ht]
  \centering
  \includegraphics[scale=0.285]{Fig003.png}
  \caption{(a) The $i^{th}$ wet element shares a common edge with the $j1^{th}$ dry element, and the bed elevation of dry element is higher than the water level at the centroid 
 	   of the $i^{th}$ element. (b) The difference between the actual and fake water level at midpoint $_{jk}$.}
  \label{fig:Fig003}
\end{figure}
 
The fluxes at the interface between element $i$ and $j1$ are calculated using the bed elevation in equation (\ref{eq:DefBatimetria}); however, fluxes at the other two element interfaces are evaluated with the actual water surface elevation $w^{in}_{jk}$ in element $i$, which drives the flow into motion in the cell $i$ and violates the well-balanced property of the scheme. The local bed modification is proposed by \cite{Liang} to tackle this problem. As shown in figure(\ref{fig:Fig003}.a), the difference between the actual and fake water level at midpoint $_{jk}$ is calculated by:

\begin{align}\label{eq:Balance}
\Delta z = \max \left( 0, B_{jk} - w^{in}_{jk} \right)
\end{align}

Then, the bed elevation and reconstructed water surface elevations at midpoint $w^{in}_{jk}$ are locally modified by subtracting $\Delta z$ from original values:

\begin{align}\label{eq:BuenBalance}
w^{in}_{jk} = w^{in}_{jk} - \Delta z
\end{align}

Once the provided reconstruction preserves the positivity of the water depth, and it is well-balanced, the computation of the velocity field $u^{in}_{jk}$ y $v^{in}_{jk}$ must 
be carried out: 

\begin{align}\label{eq:Desingularizar_u}
u^{in}_{jk} = \frac{\sqrt{2} \cdot h^{in}_{jk} \cdot hu^{in}_{jk}}{\sqrt{(h^{in}_{jk})^4 + max((h^{in}_{jk})^4,Tol)}}
\end{align}

\begin{align}\label{eq:Desingularizar_v}
v^{in}_{jk} = \frac{\sqrt{2} \cdot h^{in}_{jk} \cdot hv^{in}_{jk}}{\sqrt{(h^{in}_{jk})^4 + max((h^{in}_{jk})^4,Tol)}} 
\end{align}

The flux-discharge must be re-computed due to modifications in equation(\ref{eq:Desingularizar_u}), and \ref{eq:Desingularizar_v}: 

\begin{align} 
  hu^{in}_{jk} = h^{in}_{jk} \cdot u^{in}_{jk} \label{eq:recalculo_u} \\
  hv^{in}_{jk} = h^{in}_{jk} \cdot v^{in}_{jk} \label{eq:recalculo_v}
\end{align}

Finally, the one-sided normal velocities $w^{in}_{jk}$ y $w^{out}_{jk}$ can be computed as: 
\begin{align}
w^{in}_{jk}  &= u^{in}_{jk} \cdot n_{jk}^x +  v^{in}_{jk} \cdot n_{jk}^y \\
w^{out}_{jk} &= u^{out}_{jk} \cdot n_{jk}^x +  v^{out}_{jk} \cdot n_{jk}^y
\end{align}

Then, the local propagation speed will become:

\begin{align}\label{eq:unaDir}
a^{in}_{jk}  &=  \max \left(w^{in}_{jk} + \sqrt{g \cdot h^{in}_{jk}},w^{out}_{jk} + \sqrt{g \cdot h^{out}_{jk}},0 \right)\\
\nonumber \\
a^{out}_{jk} &= -\min \left(w^{in}_{jk} - \sqrt{g \cdot h^{in}_{jk}},w^{out}_{jk} - \sqrt{g \cdot h^{out}_{jk}},0 \right)
\end{align}

Once equipped with all the previous quantities, we can evaluate the mid-point conserved variables values at each side: \

\begin{align} \label{eq:FluxFunc} 
\; \q_{jk}  &= \left[ h_{jk} , hu_{jk}, hv_{jk} \right]^T  \\ 
\f(\q_{jk}) &= \left[ hu_{jk}, hu_{jk} + \frac{1}{2}g \, h_{jk}^2, hu_{jk} \cdot v_{jk} \right]^T \\
\g(\q_{jk}) &= \left[ hv_{jk}, hu_{jk} \cdot v_{jk}, hv_{jk} + \frac{1}{2}g \, h_{jk}^2 \right]^T 
\end{align}

The central-upwind scheme \cite{Kurganov_1} for the $j^{th}$ finite control volume $j^{th}$ must fulfill: \

\begin{align}\label{eq:CentralUpwind}
\q_j^{m+1} = \q_j^{m} & - \frac{\Delta t}{\Delta \Omega_j} \sum_{k=1}^{3} \frac{l_{jk} \cdot n_{jk}^x}{a_{jk}^{in} + a_{jk}^{out}} \Big( a_{jk}^{in} \cdot \f(\q^{out}_{jk}) + 
			   a_{jk}^{out} \cdot  \f(\q^{in}_{jk}) \Big) \nonumber \\
                      & - \frac{\Delta t}{\Delta \Omega_j} \sum_{k=1}^{3} \frac{l_{jk} \cdot n_{jk}^y}{a_{jk}^{in} + a_{jk}^{out}} \Big( a_{jk}^{in} \cdot \g(\q^{out}_{jk}) + 
			   a_{jk}^{out} \cdot  \g(\q^{in}_{jk}) \Big) \\
                      & + \frac{\Delta t}{\Delta \Omega_j} \sum_{k=1}^{3} l_{jk} \frac{ a_{jk}^{in} \cdot a_{jk}^{out}}{a_{jk}^{in} + a_{jk}^{out}} \Big( \q^{out}_{jk} -\q^{in}_{jk} \Big) + 
			  \Delta t \; \S_j(\q^{in}_j) \nonumber
\end{align}

In order to guarantee the well-balanced property, the source term must cancel all the flux terms when the lake is at rest, this means that for a given condition as $\q = [ C , 0 ,0 ]^T$ the evolution should stay the same. One can observe that when the water is at rest \q_j = \q_{jk} = [ C , 0 ,0 ]^T$, the one sided propagation speed in equation(\ref{eq:unaDir}) becomes $a_{jk}^{in} = a_{jk}^{out}$, thus the source term becomes:

\begin{align}\label{eq:Fuente}
\S_j(\q) &= \left[ \begin{array}{c}
		    0 \\
		    \displaystyle{\frac{g}{2 \cdot \Omega_j} \sum_{k=1}^{3} l_{jk} \cdot n_{jk}^x \left(w^{in}_{jk} - B_{jk} \right)^2} \\
		    \displaystyle{\frac{g}{2 \cdot \Omega_j} \sum_{k=1}^{3} l_{jk} \cdot n_{jk}^y \left(w^{in}_{jk} - B_{jk} \right)^2} \\
		    \end{array}
\right]
\end{align}

The complete process can be clearly understood referencing figure(\ref{fig:Fig004}):

\begin{figure}[ht] 
  \centering
  \includegraphics[scale=0.205]{Fig004.png}  
  \caption{First--Order Semi--Discrete Central--Upwind Method. (a) Continuous conserved variables, (b) Piece-wise constant reconstruction, (c) Positivity preserving 
	   reconstruction, (d) Well-balanced reconstruction, (e) Flux-function computation, (f) Conserved variables update.}
  \label{fig:Fig004}
\end{figure}

In order to increase the stability in time domain, the time step should be carefully chosen employing the Courant--Friedrichs--Lewy (CFL) condition, see \cite{CFL}: 

\begin{align}\label{eq:DT}
\Delta t &< \frac{1}{3} \min_{j,k} \left[ \frac{r_{jk}}{\max \left( a^{in}_{jk},a^{out}_{jk} \right) } \right]
\end{align}






<img src="https://render.githubusercontent.com/render/math?math=...">

<img src="https://render.githubusercontent.com/render/math?math=...">

<img src="https://render.githubusercontent.com/render/math?math=...">

<img src="https://render.githubusercontent.com/render/math?math=...">

<img src="https://render.githubusercontent.com/render/math?math=...">
\frac{\partial #1}{\partial t}}


Every project has a beautiful feature shocase page. It's easy to include images, in a flexible 3-column grid format. Make your photos 1/3, 2/3, or full width.

To give your project a background in the portfolio page, just add the img tag to the front matter like so:

    ---
    layout: page
    title: Project
    description: a project with a background image
    img: /assets/img/12.jpg
    ---


<div class="img_row">
    <img class="col one left" src="{{ site.baseurl }}/assets/img/1.jpg" alt="" title="example image"/>
    <img class="col one left" src="{{ site.baseurl }}/assets/img/2.jpg" alt="" title="example image"/>
    <img class="col one left" src="{{ site.baseurl }}/assets/img/3.jpg" alt="" title="example image"/>
</div>
<div class="col three caption">
    Caption photos easily. On the left, a road goes through a tunnel. Middle, leaves artistically fall in a hipster photoshoot. Right, in another hipster photoshoot, a lumberjack grasps a handful of pine needles.
</div>
<div class="img_row">
    <img class="col three left" src="{{ site.baseurl }}/assets/img/5.jpg" alt="" title="example image"/>
</div>
<div class="col three caption">
    This image can also have a caption. It's like magic.
</div>

You can also put regular text between your rows of images. Say you wanted to write a little bit about your project before you posted the rest of the images. You describe how you toiled, sweated, *bled* for your project, and then.... you reveal it's glory in the next row of images.


<div class="img_row">
    <img class="col two left" src="{{ site.baseurl }}/assets/img/6.jpg" alt="" title="example image"/>
    <img class="col one left" src="{{ site.baseurl }}/assets/img/11.jpg" alt="" title="example image"/>
</div>
<div class="col three caption">
    You can also have artistically styled 2/3 + 1/3 images, like these.
</div>


<br/><br/>


The code is simple. Just add a col class to your image, and another class specifying the width: one, two, or three columns wide. Here's the code for the last row of images above:

<div class="img_row">
    <img class="col two left" src="/img/6.jpg"/>
    <img class="col one left" src="/img/11.jpg"/>
</div>
