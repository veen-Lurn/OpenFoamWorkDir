# PhD Application - OpenFOAM Simulation Wenwen Leng

I am excited to submit my application for the [PhD Position](https://www.linkedin.com/jobs/view/doctoral-researcher-for-computational-fluid-dynamics-of-solar-energy-fed-pyrolysis-processes-for-new-fuels-at-aalto-university-3710120669/) at Aalto University in the field of Computational Fluid Dynamics for solar energy-fed pyrolysis processes. As a potential candidate, I have conducted a small 2D laminar flow simulation using OpenFOAM to demonstrate my proficiency and keen interest in this area with this Notebook.

## Laminar Flow Simulation Overview
### 1. Case Description
The simulation focuses on the laminar flow of mercury through a circular pipe with uniform heat flux walls. The key parameters of this case include a pipe radius of **0.0025 m**, a pipe length of **0.1 m,** and an average inlet velocity of **0.005 m/s**. A two-dimensional axisymmetric model was employed, and the mesh was generated using **ICEM CFD**.

The Schematic Figure is here:
[Click here to see the Sech in GitHub Page.](https://github.com/veen-Lurn/OpenFoamWorkDir/blob/main/Sche.png)

---

### 2. Media parameters

The fluid medium considered in this simulation is mercury, and its material properties are as follows:

* Density $(\rho)$ :   13529 $\mathrm{~kg} / \mathrm{m^3}$
* Viscosity: $(\mu )$: 0.001523 $\mathrm{~kg} / \mathrm{m} \cdot \mathrm{s}$
* Specific heat:  $(c_{p})$: 139.3 $\mathrm{~J} / \mathrm{kg} \cdot \mathrm{K}$
* Thermal conductivity: $(\kappa)$: 8.54 $\mathrm{~W} / \mathrm{m} \cdot \mathrm{K}$

### 3. Boundary Conditions
The boundary conditions applied in the simulation are as follows:
- Entrance condition: Fully developed speed with an average velocity of $u_m=0.005 \mathrm{~m} / \mathrm{s}$,
  where, the fully developed speed distribution:

$$
v=2 u_m\left(1-\frac{r^2}{R^2}\right)
$$

* Inlet temperature $(T_i)$: $300 \mathrm{~K}$
* Wall heat flux: $(q_s^{\prime \prime})$: $5000 \mathrm{~W} / \mathrm{m}^{2}$

### 4. Analytical Calculation Method

Theoretical expressions were used for pressure drop and heat transfer calculations. The pressure drop is calculated using the following formula.

1. Pressure drop calculation

The theoretical values come from the [Fundamentals of Heat and Mass Transfer](https://mech.at.ua/HeatandMassTransfer7thEdition-Incropera-dewitt.pdf):

> [Theodore L. Bergman, Adrienne S. Lavine, Frank P. Incropera, David P. DeWitt. Fundamentals of Heat and Mass Transfer [7ed.]. P523.](https://mech.at.ua/HeatandMassTransfer7thEdition-Incropera-dewitt.pdf)

The fully developed velocity expression and head loss expression are given. They are:

$$
u=2 u_m\left(1-\frac{r^2}{R^2}\right)
$$

$$
\Delta p=\frac{32 \mu L u_m}{d^2}
$$

where $u_m$ is the average flow velocity, $\mu$ is the hydrodynamic viscosity, and $d$ is the pipe diameter.

2. Heat transfer calculation

For heat transfer, the temperature distribution along the pipe is determined by the P530 in [Fundamentals of Heat and Mass Transfer](https://mech.at.ua/HeatandMassTransfer7thEdition-Incropera-dewitt.pdf)  :

$$
T_m(x)=T_{m, i}+\frac{q_s^{\prime \prime} \pi d}{\dot{m} c_p} x
$$

### 5. OpenFOAM Simulation Details

In order to perform the OpenFOAM simulation, the following modifications were made:
#### 5.1 Modification of Solver (icoTempFoam)
I realized that the icoFoam solver lacks temperature solving capabilities. To address this, the  <font color=red>**`solver`**</font> was modified as follows: 
- Added thermal diffusion coefficient to the  <font color=red> **`createFields.H`**</font> file. 



- Introduced the temperature field <font color=red>**`T`**</font> in the solver. 
- Included the temperature field control equation in <font color=red>**`icoTempFoam.C`**</font>. 
- Compiled the modified solver using the <font color=red>**`wmake`**</font> command. 

> [Note: For this part, please refer to:
> http://openfoamwiki.net/index.php/How_to_add_temperature_to_icoFoam](http://openfoamwiki.net/index.php/How_to_add_temperature_to_icoFoam)


#### 5.2 Grid Preparation
The <font color=red>**`cavity`** </font>  case was used as a template, and a **Fluent Cas** file was imported to process the grid. The aim was to convert a 2D mesh to an axisymmetric mesh, although there were challenges in compiling the  <font color=red>**`makeAxialMesh`** </font> tool.

In this case, I try to use the official **OpenFOAM** calculation example   <font color=red> **`cavity`**</font> as a template, and then use the importing **Fluent Cas** files to process the grid.

* Preparing Files, and Place the grid file  <font color=red> `laminar.cas`</font> into the<font color=red> `laminar`</font> folder.
  
  The calculation example file structure is shown in the figure below.
[Click here to see the file structure PrtSc in GitHub Page.](https://github.com/veen-Lurn/OpenFoamWorkDir/blob/main/9c64f7100191c48c295fa5418f6715b.png)


Here, the function in **ICEM CFD** is used <font color=red> **`Extrude Mesh`**</font>  to process the 2D grid and convert it into a three-dimensional structure, as shown in the figure below (the **OUTLET** boundary is not marked in the figure).

[Click here to see the Mesh in GitHub Page.](https://github.com/veen-Lurn/OpenFoamWorkDir/blob/main/9c64f7100191c48c295fa5418f6715b.png)

After **ICEM CFD** processing is completed, the calculation grid is exported. 
Then I try to run the following command to convert the mesh for OpenFoam:

        fluentMeshToFoam laminar.msh  
	    checkMesh

After the grid conversion is completed and there are no problems, open the file `constant/polyMesh/boundary`to modify the boundary and specify `SIDE1` and `SIDE2` the boundary type of the boundary `wedge`. 

> **Note**: In order to convert a 2D mesh to an axisymmetric mesh, one need to use the unofficial tool `makeAxialMesh`, see，[https://github.com/cfdcoach/MakeAxialMesh.git](https://github.com/cfdcoach/MakeAxialMesh.git)
> But I have never been able to compile this tool successfully.

#### 5.3 Setting Material Properties

The kinematic viscosity and thermal diffusion coefficient were specified in the  <font color=red>**`constant/transportProperties`** </font>  file according to the given fluid properties.

The kinematic viscosity needs to be specified:
$$
\nu=\frac{\mu}{\rho}=\frac{0.001523}{13529}=1.12573 \times 10^{-7} \mathrm{~m}^2 / \mathrm{s}
$$
The thermal diffusion coefficient DT needs to be specified in this file, which is defined as:
$$
\mathrm{DT}=\frac{k}{\rho c_p}=\frac{8.51}{13529 \times 139.3}=4.51557 \times 10^{-6} \mathrm{~m}^2 / \mathrm{s}
$$

#### 5.4 Boundary Conditions and Initial Conditions

 - The <font color=red>**`U`**  </font> and <font color=red>**`p`**  </font>  files were modified to accommodate fully developed flow conditions at the inlet. 

 - The  <font color=red>**`T`**  </font>  file was created and set to handle the specified wall heat flux using Fourier's law, AND the conversion gives:
$$
\nabla T=\frac{q}{k}=\frac{5000}{8.54}=585.48
$$
#### 5.5 Solution Control

Settings in the <font color=red>**`controlDict`**  </font> <font color=red>**`fvSchemes`**  </font>, and <font color=red>**`fvSolution`**  </font> files were adjusted to ensure appropriate control of the simulation. These files could be found in my attached.

### 6. Calculation Results

#### 6.1 Speed Distribution at 30 Seconds    

 [Click here to get the result of the speed distribution in GitHub Page.](https://github.com/veen-Lurn/OpenFoamWorkDir/blob/main/Speed%20Distribution.png)

#### 6.2  Temperature Distribution at 30 Seconds
[Click here to get the result of the Temp distribution in GitHub Page.](https://github.com/veen-Lurn/OpenFoamWorkDir/blob/main/Temperature%20Distribution.png)

#### 6.3 Exit Position Velocity Distribution at 30 Seconds
[Click here to get the result of the Exit Position Velocity in GitHub Page.](https://github.com/veen-Lurn/OpenFoamWorkDir/blob/main/Exit%20Position%20Velocity%20Distribution.png)

#### 6.4 Outlet Axis Temperature Statistics
The outlet axis temperature is statistically calculated, as shown in the figure below, and it is measured at **337.494 K**. This value closely aligns with the literature value of **341 K**.

[Click here to get the result of the Outlet Axis Temperature in GitHub Page.](https://github.com/veen-Lurn/OpenFoamWorkDir/blob/main/Outlet%20Axis%20Temperature.png)

#### 6.5 Pressure Drop Calculation
To calculate the pressure drop, I created a slice and then an integrate variable on **Slice1**. Following this, I created **Calculator1** on **IntegrateVariables1** to obtain the average pressure on the surface. The average pressure on the inlet surface is shown in the figure below, with a pressure value of **7.21232e-05** $\mathrm{m^2}/\mathrm{s^2}$.
[Click here to get the result of the  Pressure Drop in GitHub Page](https://github.com/veen-Lurn/OpenFoamWorkDir/blob/main/The%20average%20pressure%20on%20the%20inlet%20surface.png)

The outlet static pressure is set to **0** in the boundary conditions, so there is no need to obtain it through a calculator.

It's important to note that the pressure calculated by OpenFOAM in incompressible flow is the kinematic pressure, expressed as the quotient of the pressure value and the density value. To obtain the density value, the obtained kinematic pressure value can be multiplied by the density value. Therefore, the pressure drop at the inlet and outlet can be calculated as:
$$
\Delta p=7.21232 \times 10^{-5} \times 13512=0.9745286784 \mathrm{~Pa}
$$
The theoretical pressure drop is calculated as:
$$
\Delta p=\frac{32 \mu L u_m}{d^2}=\frac{32 \times 0.001523 \times 0.1 \times 0.005}{0.005^2}=0.97472 \mathrm{~Pa}
$$

### 7. Verification Results
Through the previous calculations, a comparison between the calculated results and the theoretical values can be obtained, as shown in the table below.

| Physical Quantity | Target Value | OpenFOAM Calculation Results | Relative Error |
|:------------------:|:------------:|:----------------------------:|:--------------:|
|   **Temperature**(**K**)   |      341     |            337.494           |    -1.0282%    
| **Pressure drop** (**Pa**) |    0.97472   |          0.9745286784        |    +0.0196%    |

The results indicate a close alignment between the calculated and theoretical values, affirming the accuracy of the simulation.

##  Summary 

In this simulation, I present a 2D laminar flow simulation conducted using OpenFOAM for the first time rather than using traditional commercial software to showcase my proficiency and genuine passion for the field.

I try to modify the icoFoam solver, delving into the intricacies of temperature solving, and embracing the challenges of grid conversion. The transformation of a 2D mesh into an axisymmetric masterpiece was a humbling yet rewarding experience.

The results of this simulation include temperature and speed distributions, pressure drop calculations, and a careful verification against theoretical values. I'm genuinely pleased to see that the results align closely, showcasing the precision of the simulation.

This endeavor not only demonstrates my growing technical competence in Computational Fluid Dynamics but also reflects my deep-rooted passion for the art of scientific exploration. I'm eager to bring this enthusiasm and my unique perspective to Aalto University, where I hope to contribute to your esteemed research program.

Thank you for taking the time to consider my application.

Warm regards,

M.Sc. Wenwen Leng

