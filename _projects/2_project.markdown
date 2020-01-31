---
layout: page
title: A GPU implementation of the Shallow Water Equations.
description: How to speedup the Shallow Water Equations on Graphic processors.
img: /assets/img/2.jpg
---

This project is  basically a numerical implementation on GPU of the Shallow Water Equation taken form [[1]](#1) and [[4]](#4). The chosen method employs a very efficient finite volume algorithm for the numerical solution of the ***Shallow Water Equations***, 
here SWE, which has not only the characteristic to be robust, and accurate, but also it is implemented in non-structured triangular mesh.

In general, method that solves the SWE must fulfill both the well-balanced property (this means must not generate synthetic perturbations for lake at rest conditions), and must preserve the positivity preserving property for the water depth, this means the water column must be always positive [[1]](#1). 

Because of the previous reasons, an improved first-order central-upwind was chosen to be implemented, see [[2]](#2), which in contrast to their predecessors, is able to preserve the lake at rest conditions and the positivity water level [[3]](#3), employing a variable time step to increase the convergence in time. Moreover, in order to increase the stability of the method, it is necessary to increase the resolution of the mesh in coastal areas. This is carried out by employing a non-structured mesh. To fulfill these equirements triangular elements were employed.


### Model Definition.
In this section, it will be defined the main variables that are going to be employed during this document. We will use normal variables $$q$$ as scalar, while in bold $$\mathbf{q}$$ as vectors, i.e., we will have $$\mathbf{q} = (q_1, q_2, q_3)^\top$$ as a vector of conserved variables.  

![image-title-here](/assets/img/Project2/Fig001.png){:class="img-responsive"}{:height="" width="700px"}
*Figure 1: Main variables (states) of the shallow water equations.*

where the variables represented in the figure are: $$w(x,y,t)$$ the surface elevation,  measured from mean sea level to the water level. $$h(x,y,t)$$ the water depth which is measured from the bed elevation to the water surface. $$u(x,y,t)$$ the velocity field along $$x$$-direction, and $$v(x,y,t)$$ the velocity field along $$y$$-direction. $$B(x,y)$$ represents the bathymetry. It is measured from the mean sea level to the bottom floor. In addition, there are other associated variables $$hu(x,y,t)$$ is the flux-discharge along $$x$$-direction, and $$hv(x,y,t)$$ is the flux-discharge along $$y$$-direction.

### The Shallow Water Equations.

Assuming hydrostatic pressure condition, the SWE are obtained by integrating the Navier-Stokes equations over the water depth. A system of bi-dimensional equations is obtained, where the horizontal velocities are an average of the velocity along the water column. Neglecting the kinematic and turbulent terms, the SWE can be written as: 

$$
h_t + (hu)_x + (hv)_y = 0 \,,
$$

$$ 
(hu)_t + \left(hu^2 + \frac{1}{2} g h^2 \right)_x + (huv)_y    = -g h B_x - g n^2 \frac{(hu) \sqrt{(hu)^2 + (hv)^2}}{h^{7/3}} \,,
$$

$$
(hv)_t + (huv)_x  + \left( hv^2 + \frac{1}{2} g h^2 \right)_y  = -g h B_y - g n^2 \frac{(hv) \sqrt{(hu)^2 + (hv)^2}}{h^{7/3}} \,,
$$

The first equation represents the mass balance due to the change of water height of the water column in a certain point, and the last two represent the moment balance due to the change in the discharge with the weight of the eater column, the bathymetry source, and the bottom friction's force, etc. This set of equations can be written down on its conserved vector form as follows: 

$$
\frac{\partial \mathbf{q}}{\partial t} + \frac{\partial \mathbf{f}(\mathbf{q}) }{\partial x} + \frac{\partial \mathbf{g}(\mathbf{q}) }{\partial y} = \mathbf{S}(\mathbf{q}) + \mathbf{R}(\mathbf{q}) \,,
$$

where the previous vector variables are:

$$
\mathbf{q}  = \left[ h,  hu,  hv \right]^\top \,,
$$

$$
\mathbf{f}(\mathbf{q}) = \left[ hu, hu + \frac{1}{2}gh^2, huv \right]^\top \,,
$$

$$
\mathbf{g}(\mathbf{q}) = \left[ hv, huv, hv + \frac{1}{2}gh^2 \right]^\top \,,
$$

$$
\mathbf{S}(\mathbf{q}) = \left[ 0, -gh \frac{\partial B}{\partial x}, -gh \frac{\partial B}{\partial y} \right]^\top \,,
$$

$$
\mathbf{R}(\mathbf{q}) = \left[ 0, g n^2 \frac{(hu) \sqrt{(hu)^2 + (hv)^2}}{h^{7/3}}, g n^2 \frac{(hv) \sqrt{(hu)^2 + (hv)^2}}{h^{7/3}} \right]^\top \,,
$$

and $$\mathbf{S}(\mathbf{q})$$ represents the source term, $$\mathbf{R}(\mathbf{q})$$ represents the bottom friction term, and $$n$$ is the Manning's roughness coefficient.


### The Finite Volume Method.

The finite volume method is employed in order to solve the SWE. First, the flux-field will be written in its vector form as: $$\overrightarrow{\mathbf{F}}(\mathbf{q}) = (\mathbf{f},\mathbf{g})$$. Then, we have

$$
\frac{\partial \mathbf{q}}{\partial t} + \nabla \cdot \overrightarrow{\mathbf{F}}(\mathbf{q}) = \mathbf{S}(\mathbf{q}) + \mathbf{C}(\mathbf{q}) + \mathbf{R}(\mathbf{q}) \,.
$$

Integrating by part over a finite triangular control volume $$\Omega$$, we get: 
 
$$
\iint_{\Omega} \frac{\partial \mathbf{q}}{\partial t} \; d \Omega + \iint_{\Omega} \nabla \cdot \overrightarrow{\mathbf{F}}(\mathbf{q}) \; d \Omega= \iint_{\Omega} \left( \mathbf{S}(\mathbf{q}) + \mathbf{C}(\mathbf{q}) + \mathbf{R}(\mathbf{q}) \right) \; d \Omega \,.
$$

Applying the Stokes' theorem to a finite triangular element, we obtain:

$$
\frac{\partial \mathbf{q}}{\partial t} \cdot \Omega + \oint_{\Gamma} \mathbf{F}(\mathbf{q}) \cdot \overrightarrow{n} \; d \Gamma = \mathbf{S}(\mathbf{q}) \cdot \Omega + \mathbf{C}(\mathbf{q}) \cdot \Omega + \mathbf{R}(\mathbf{q}) \cdot \Omega \,,
$$

where $$\Omega$$ is the control volume, $$\Gamma$$ is the boundary of the control volume, $$\overrightarrow{n}$$ is the outward normal vector on the side, and $$\mathbf{F}(\mathbf{q})$$ is the normal flux to the edge of each side of the control volume. 

Employing the Euler's method to discretize the temporal variable, we get:

$$
\mathbf{q}_j^{m+1} = \mathbf{q}_j^{m} - \frac{\Delta t}{\Delta \Omega_j} \sum_{k=1}^{3} \mathbf{F}_{jk}(\mathbf{q}^{m}) \cdot n_{jk} \cdot l_{jk} + \Delta t \cdot \mathbf{S}_j(\mathbf{q}^{m}) \,,
$$

where $$n_{jk}$$ is the normal vector to the $$k^{th}$$ mid-point side of the $$j^{th}$$ triangle, $$\Delta t$$ is the time step, $$\Delta \Omega_j$$ is the control volume for the $$j^{th}$$ triangle, $$l_{jk}$$ is the length of the $$k^{th}$$ side of the $$j^{th}$$ triangle, and $$\mathbf{q}_j^{m}$$ is the vector of conserved variables for the $$j^{th}$$ triangle at the $$m^{th}$$ time step.

Terms such as the bottom friction and Coriolis' force can be incorporated employing a semi-implicit discretization: 

$$
\mathbf{C}(\mathbf{q}) + \mathbf{R}(\mathbf{q}) \approx  H(\mathbf{q}^{m+1}) \cdot \mathbf{q}^{m+1} \,.
$$

In this regard, the friction and the Coriolis terms can be incorporated correcting the vector of conserved variables: 

$$
\mathbf{q}_j^{m+1} = \frac{\mathbf{q}_j^{m+1}}{1 + \Delta t \cdot H(\mathbf{q}_j^{m+1})} \,.
$$

### First--Order Semi--Discrete Central Upwind.

The **First--order semi--discrete central--upwind** method is employed to solve the discrete form of the Sallow Water Equations. we use as a reference the scheme depicted in the figure bellow: 

![image-title-here](/assets/img/Project2/Fig002.png){:class="img-responsive"}{:height="" width="700px"}
*Figure 2: (a) Finite volume mesh, (b) typical control volume and its associated variables.*

The method starts by defining a piece-wise constant reconstruction of the bathymetry. The mass center value of the elements will be taken as constant over the element. Therefore, a discontinuous values will be provided at each side of the element, this is ``in`` and the ``out`` interface as it is shown in the latter figure. In order  to obtain the final states of the conserved variables, a single value of bed elevation suggested by [[5]](#5) is defined:

$$
 B_{jk} = \max \left( B^{in}_{jk},B^{out}_{jk} \right) \,.
$$

The **First--Order Semi--Discrete Central--Upwind Method** begins by knowing the explicit functions of the conserved variables at the mass center of each triangle at the previous time step, i.e., $$\mathbf{q}_j^m$$. Then, a piece-wise constant reconstruction will be employed, this is, the conserved variable values at the mid-point of each side are the same as the mass center. 

$$
\mathbf{q}_{jk}^{in} = \mathbf{q}_j^{m} \,.
$$

The water level surface reconstruction must be so that the water depth has to be always positive; therefore, the reconstructed water depth at the midpoint $$h_{jk}$$ should be re-defined as: 

$$
h^{in}_{jk} = \max \left(Tol, w^{in}_{jk} - B_{jk} \right) \,.
$$

It is simple to show that the reconstruction process defined above do not affect the well-balanced property of the numerical scheme when only wet-bed applications are simulated. However, for dry-bed applications, the well-balanced property is violated, see the following figure

![image-title-here](/assets/img/Project2/Fig003.png){:class="img-responsive"}{:height="" width="700px"}
*Figure 3: (a) The $$i^{th}$$ wet element shares a common edge with the $$j1^{th}$$ dry element, and the bed elevation of dry element is higher than the water level at the centroid of the $$i^{th}$$ element. (b) The difference between the actual and fake water level at midpoint $$_{jk}$$.*
 
The fluxes at the interface between element $$i$$ and $$j1$$ are calculated using the bed elevation; however, fluxes at the other two element interfaces are evaluated with the actual water surface elevation $$w^{in}_{jk}$$ in element $$i$$, which drives the flow into motion in the cell $$i$$ and violates the well-balanced property of the scheme. The local bed modification is proposed by [[4]](#4) to tackle this problem. As shown in last figure, the difference between the actual and fake water level at midpoint $$_{jk}$$ is calculated by:

$$
\Delta z = \max \left( 0, B_{jk} - w^{in}_{jk} \right) \,.
$$

Then, the bed elevation and reconstructed water surface elevations at midpoint $$w^{in}_{jk}$$ are locally modified by subtracting $$\Delta z$$ from original values: 

$$
w^{in}_{jk} = w^{in}_{jk} - \Delta z
$$

Once the provided reconstruction preserves the positivity of the water depth, and it is well-balanced, the computation of the velocity field $$u^{in}_{jk}$$ and $$v^{in}_{jk}$$ must be carried out: 

$$
u^{in}_{jk} = \frac{\sqrt{2} \cdot h^{in}_{jk} \cdot hu^{in}_{jk}}{\sqrt{(h^{in}_{jk})^4 + max((h^{in}_{jk})^4,Tol)}}
$$

$$
v^{in}_{jk} = \frac{\sqrt{2} \cdot h^{in}_{jk} \cdot hv^{in}_{jk}}{\sqrt{(h^{in}_{jk})^4 + max((h^{in}_{jk})^4,Tol)}} 
$$

The flux-discharge must be re-computed due to modifications performed to $u$ and $v$ velocities: 

$$
hu^{in}_{jk} = h^{in}_{jk} \cdot u^{in}_{jk} \,,
$$

$$
hv^{in}_{jk} = h^{in}_{jk} \cdot v^{in}_{jk} \,.
$$

Finally, the one-sided normal velocities $$w^{in}_{jk}$$ and $$w^{out}_{jk}$$ can be computed as

$$
w^{in}_{jk}  = u^{in}_{jk} \cdot n_{jk}^x +  v^{in}_{jk} \cdot n_{jk}^y \,,
$$

$$
w^{out}_{jk} = u^{out}_{jk} \cdot n_{jk}^x +  v^{out}_{jk} \cdot n_{jk}^y \,.
$$

Then, the local propagation speed becomes:

$$
a^{in}_{jk}  =  \max \left(w^{in}_{jk} + \sqrt{g \cdot h^{in}_{jk}},w^{out}_{jk} + \sqrt{g \cdot h^{out}_{jk}},0 \right) 
$$

$$
a^{out}_{jk} = -\min \left(w^{in}_{jk} - \sqrt{g \cdot h^{in}_{jk}},w^{out}_{jk} - \sqrt{g \cdot h^{out}_{jk}},0 \right)
$$

Once equipped with all the previous quantities, we can evaluate the mid-point conserved variables values at each side: 

$$
\mathbf{q}_{jk}  = \left[ h_{jk} , hu_{jk}, hv_{jk} \right]^\top \,,
$$

$$
\mathbf{f}(\mathbf{q}_{jk}) = \left[ hu_{jk}, hu_{jk} + \frac{1}{2}g \, h_{jk}^2, hu_{jk} \cdot v_{jk} \right]^\top \,,
$$

$$
\mathbf{g}(\mathbf{q}_{jk}) = \left[ hv_{jk}, hu_{jk} \cdot v_{jk}, hv_{jk} + \frac{1}{2}g \, h_{jk}^2 \right]^\top \,. 
$$

The central-upwind scheme [[1]](#1) for the $$j^{th}$$ finite control volume $$j^{th}$$ must fulfill: 

$$
\mathbf{q}_j^{m+1} = \mathbf{q}_j^{m} - \frac{\Delta t}{\Delta \Omega_j} \sum_{k=1}^{3} \frac{l_{jk} \cdot n_{jk}^x}{a_{jk}^{in} + a_{jk}^{out}} \left( a_{jk}^{in} \cdot \mathbf{f}(\mathbf{q}^{out}_{jk}) + a_{jk}^{out} \cdot  \mathbf{f}(\mathbf{q}^{in}_{jk}) \right) -
$$

$$
\frac{\Delta t}{\Delta \Omega_j} \sum_{k=1}^{3} \frac{l_{jk} \cdot n_{jk}^y}{a_{jk}^{in} + a_{jk}^{out}} \left( a_{jk}^{in} \cdot \mathbf{g}(\mathbf{q}^{out}_{jk}) + a_{jk}^{out} \cdot  \mathbf{g}(\mathbf{q}^{in}_{jk}) \right) +
$$

$$
\frac{\Delta t}{\Delta \Omega_j} \sum_{k=1}^{3} l_{jk} \frac{ a_{jk}^{in} \cdot a_{jk}^{out}}{a_{jk}^{in} + a_{jk}^{out}} \left( \mathbf{q}^{out}_{jk} -\mathbf{q}^{in}_{jk} \right) + \Delta t \; \mathbf{S}_j(\mathbf{q}^{in}_j)
$$

In order to guarantee the well-balanced property, the source term must cancel all the flux terms when the lake is at rest, this means that for a given condition as $$\mathbf{q} = [ C , 0 ,0 ]^\top$$ the evolution should stay the same. One can observe that when the water is at rest $$\mathbf{q}_j = \mathbf{q}_{jk} = [ C , 0 ,0 ]^\top$$, the one sided propagation speed becomes $$a_{jk}^{in} = a_{jk}^{out}$$, thus the source term becomes:

$$
\mathbf{S}_j(\mathbf{q}) = \left[ \begin{array}{c} 0 \\ \displaystyle{\frac{g}{2 \cdot \Omega_j} \sum_{k=1}^{3} l_{jk} \cdot n_{jk}^x \left(w^{in}_{jk} - B_{jk} \right)^2} \\ \displaystyle{\frac{g}{2 \cdot \Omega_j} \sum_{k=1}^{3} l_{jk} \cdot n_{jk}^y \left(w^{in}_{jk} - B_{jk} \right)^2} \end{array} \right]
$$

The complete process can be clearly understood referencing figure(\ref{fig:Fig004}): 

![image-title-here](/assets/img/Project2/Fig004.png){:class="img-responsive"}{:height="" width="700px"}
*Figure 4: First--Order Semi--Discrete Central--Upwind Method. (a) Continuous conserved variables, (b) Piece-wise constant reconstruction, (c) Positivity preserving reconstruction, (d) Well-balanced reconstruction, (e) Flux-function computation, (f) Conserved variables update.*

In order to increase the stability in time domain, the time step should be carefully chosen employing the Courant--Friedrichs--Lewy (CFL) condition, see [[6]](#6):

$$
\Delta t < \frac{1}{3} \min_{j,k} \left[ \frac{r_{jk}}{\max \left( a^{in}_{jk},a^{out}_{jk} \right) } \right] \,.
$$

### SWE Algorithm.
The main structure of the program is summarized in algoritm (\ref{alg:CentralUpwind}): \\

\begin{algorithm}[!hbt]
  \begin{algorithmic}[1]
     \REQUIRE \texttt{Inputs}: 
	       \begin{itemize}
                \item Total time of simulation: $t_{max}$ [s]. 
   		%\item Time sampling: $\Delta t_{save}$ [s]
                %\item Manning's coefficient: $n$ [1]. 
                \item Maximum allowed tolerance: $Tol$ [m]. 
                %\item Constant of gravity: $g$ [$m/s^2$].  
                %\item Elements array: $ElemNodes$
                %\item Neighbor array: $ElemNeigh$.
                %\item Coordinate array: $ElemCoords$.
                \item Bathymetry values: $B_{jk}$.
                \item Initial condition: $w_{jk}^0$, $hu_{jk}^0$, $hv_{jk}^0$.
               \end{itemize}
      %\vspace{1.5mm}
      \REQUIRE \texttt{Process}: 
      	       \begin{itemize}
                 \item Compute the area of each triangle: $\Omega_{j}$. 
                 \item Compute the length of each sides: $l_{jk}$. 
		 \item Compute the altitude of each sides: $r_{jk}$.
                 \item Compute the normal vectors: $\overrightarrow{n}_{jk}$. 
               \end{itemize}
      %\vspace{1.5mm}
      \WHILE {$t \leq t_{max}$}
      %\vspace{1.0mm}
	  \STATE Piece-wise constant Reconstruction $\mathbf{q}^{(1)} \leftarrow \eta_{jk}$, use (\ref{eq:Reconstruccion})
	  \STATE Impose water depth positivity preserving, use (\ref{eq:Profundidad})
	  \STATE Flux-discharge reconstruction $\mathbf{q}^{(2)} \leftarrow hu_{jk}$, $\mathbf{q}^{(3)} \leftarrow hv_{jk}$, use (\ref{eq:Reconstruccion})
	  \STATE Desingularize the velocities $u_{jk}$, $v_{jk}$, use (\ref{eq:Desingularizar_u}), (\ref{eq:Desingularizar_v})
	  \STATE Re-compute the flux-discharge $hu_{jk}$, $hv_{jk}$, use (\ref{eq:recalculo_u}), (\ref{eq:recalculo_v})
	  \STATE Compute the one-sided speed velocities $a^{in}_{jk}$, $a^{out}_{jk}$, use (\ref{eq:unaDir})
	  \STATE Compute the maximum allowed time step $\Delta t$,  use (\ref{eq:DT})
	  \STATE Evaluate the source term $\mathbf{S}_j$,  use (\ref{eq:Fuente})
	  \STATE Evaluate the friction term $\R_j$,  use (\ref{eq:Fuente})
	  \STATE Evaluate the flux functions $\mathbf{f}(\mathbf{q}_{jk})$, $\g(\mathbf{q}_{jk})$, use (\ref{eq:FluxFunc})
	  \STATE Update the state variables $\mathbf{q}_j$,  use (\ref{eq:CentralUpwind})
	  \STATE Correct the states variables $\mathbf{q}_j$,  use (\ref{eq:update})
          \STATE Impose boundary conditions.
	  %\vspace{1.0mm}
      \ENDWHILE
  \end{algorithmic}
  \caption{First--Order Well-Balanced Positivity Preserving Central-Upwind}\label{alg:CentralUpwind}
\end{algorithm}

### Numerical Examples.

#### Solitary Wave On A Simple Beach.
Figure(\ref{fig:Fig005}) provides a schematic of the experiments indicating the parameters employed:

\begin{figure}[!ht]
\centering \includegraphics[scale=0.225]{Fig005.png}
\caption{Definition sketch of solitary wave run-up on a plane beach.}
\label{fig:Fig005}
\end{figure}

The defined parameters in figure (\ref{fig:Fig006}) are, $H$: the incident solitary wave height, $R$: the run-up, $x_1$: represents the initial position of the 
wave-crest, $x_0$: is the starting point of the sloping beach, $d$: is the constant bathymetry depth, and $g$: the gravitational constant. \\

Here, we have considered a $80 \; [m]$ long channel with soft conditions at both ends i.e, $x = -10 \; [m]$, and $x = 70 \; [m]$. Periodic condition were applied 
for the upper and lower boundaries i.e $y = -5 \; [m]$, and $y = 5 \; [m]$. We use the parameters $x_0 = 19.85 \; [m]$, $x_1 = 37.35 \; [m]$, $d = 1.00 \; [m]$, 
the gravitational constant was taken as $g = 9.80 \; [m/s^2]$. Surface roughness becomes important for runup over harsh slopes and a Manning's coefficient 
$n = 0.01$ describes the surface condition of the smooth glass beach in the laboratory experiments. \\

The animation should look like:

\begin{figure}[!htb]
     \begin{center}
        \subfigure{
            \includegraphics[width=0.375\textwidth]{Run001.png}
        }
	\subfigure{
            \includegraphics[width=0.375\textwidth]{Run002.png}
        } \\ %  ------- End of the first row ----------------------%
        \subfigure{
            \includegraphics[width=0.375\textwidth]{Run003.png}
        } 
        \subfigure{
            \includegraphics[width=0.375\textwidth]{Run004.png}
        }
    \end{center}
    \label{fig:fig407_02}
\end{figure}

#### Solitary Wave On A Conical Island.
Figure(\ref{fig:Fig006}) shows a schematic sketch of the experiment. The basin is $25 \; [m]$ long and $30 \; [m]$ wide. The circular 
island has the shape of a truncated cone with diameters of $7.2 \; [m]$ at the base and $2.2 \; [m]$ at the crest. The island is $0.625 \; [m]$ high 
and has a side slope of $1:4$. \\

\begin{figure}[!ht]
\centering \includegraphics[scale=0.350]{Fig006.png}
\caption{Schematic sketch of the conical island experiment: (a) plane view and (b) side view.}
\label{fig:Fig006}
\end{figure}

Here, we consider a rectangular domain of $[-17.5,17.5] \times [-6.25,6.25]$ with soft conditions at both ends i.e., $x = -17.5 \;[m]$, and $x = 17.5 \; [m]$. Periodic 
condition were applied for the upper and lower boundaries i.e., $y = -6.25 \; [m]$, and $y = 6.25 \; [m]$. We use the parameters $x_1 = 12.96 \; [m]$ as the starting 
point of the wave height, the gravitational constant was taken as $g = 9.80 \; [m/s^2]$. The experiment also covers the water depths $d = 0.32 \; [m]$  and the solitary 
wave heights $H = 0.0144 \; [m]$. Therefore, the solitary wave is generated from the left boundary, and a Manning's roughness coefficient was set to be $n = 0.016$ for 
the smooth concrete finish. \\

The animation should look like:

\begin{figure}[!htb]
     \begin{center}
        \subfigure{
            \includegraphics[width=0.375\textwidth]{Island001.png}
        }
	\subfigure{
            \includegraphics[width=0.375\textwidth]{Island002.png}
        } \\ %  ------- End of the first row ----------------------%
        \subfigure{
            \includegraphics[width=0.375\textwidth]{Island003.png}
        } 
        \subfigure{
            \includegraphics[width=0.375\textwidth]{Island004.png}
        }
    \end{center}
    \label{fig:fig407_02}
\end{figure}


### References
<a id="1">[1]</a> 
Bollermann, Andreas and Chen, Guoxian and Kurganov, Alexander and Noelle, Sebastian
A Well-Balanced Reconstruction of Wet/Dry Fronts for the Shallow Water Equations.
Journal of Scientific Computing, August 2013, (56) 267--290.	

<a id="1">[2]</a> 
Bryson, S. and Levy, D.
Balanced Central Schemes for the Shallow Water Equations on Unstructured Grids. 
SIAM Journal on Scientific Computing, 2005, (27) 532--552.

<a id="1">[3]</a> 
Bryson, Steve and Epshteyn, Yekaterina and Kurganov, Alexander and Petrova, Guergana 
Well-balanced positivity preserving central-upwind scheme on triangular grids for the Saint-Venant system. 
ESAIM: Mathematical Modelling and Numerical Analysis, 2011, (54) 423--446

<a id="1">[4]</a> 
Q. Liang
Flood simulation using a well-balanced shallow flow model.
Journal of Hydraulic Engineering, 2010, (136) 669--675

<a id="1">[5]</a> 
E. Audusse and  F. Bouchut and  M. Bristeau and R. Klein and B. Perthame.
A fast and stable well-balanced scheme with hydrostatic reconstruction for shallow water flows.
SIAM Journal on Scientific Computing, 2004, (25) 2050--2065

<a id="1">[6]</a> 
Courant R. and Friedrichs K. and Lewy H.
textit{On the partial difference equations of mathematical physics.
IBM Journal of Research and Development, 1967, (11) 215--234