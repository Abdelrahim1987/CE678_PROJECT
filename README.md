# CE678_PROJECT
 Geoid modelling ce678 project
## REGIONAL GEOID MODELLING
### MEMBERS
1. Jainam
2. Vyush
3. Sachin

## How to use
This is a guide on how to use this software
We use classical remove-restore method to find geoid height in this code.

If stuck anywhere use this guide or type help corresponding to the function in Matlab.

There is a also a file `project.ipynb` that we have created to demonstrate our code on Nebraska,USA.

Also `project.pdf` is the project report file.

### Step 1

1. Download the airborne gravity data from [here](https://geodesy.noaa.gov/GRAV-D/data_products.shtml) by choosing any state tile.

2. Use the [`import_airborne_gravity_data.m`](https://github.com/Jainam-IITK/CE678_PROJECT/blob/main/import_airborne_gravity_data.m) function to import the data from the file of gravity data.

3. Save the matrix into vectors . Ex - 

   T = import_airborne_gravity_data('file_path');`

   `latitude = T(:,1);`

   `longitude = T(:,2);`

   `elipsoidal_height = T(:,4);`

   `gravity_filtered = T(:,3);`

### Step 2

1. Download the geoid heights of GGM model from [here](http://icgem.gfz-potsdam.de/calcgrid)

2. You can import them with [import_geoid_heights](https://github.com/Jainam-IITK/CE678_PROJECT/blob/main/import_geoid_heights.m)

3. Code can be written as - 

   `nggm = import_geoid_heights('filepath',latitude,longitude)`

4. Calculate the **Orthometric height** by adding `nggm` to `ellisoidal_height`  using this function-

   [`orthometric_height = calc_orthometric_height(elipsoidal_height,nggm)`](https://github.com/Jainam-IITK/CE678_PROJECT/blob/main/calc_orthometric_height.m)

### Step 3

Calculate the free air anomaly using the formula
<!-- $$
\Delta g=\left(g_{\text {observed }}+\delta g_{\text {Free air }}\right)-\gamma
$$ --> 

<div align="center"><img src="https://render.githubusercontent.com/render/math?math=%5CDelta%20g%3D%5Cleft(g_%7B%5Ctext%20%7Bobserved%20%7D%7D%2B%5Cdelta%20g_%7B%5Ctext%20%7BFree%20air%20%7D%7D%5Cright)-%5Cgamma"></div>
Code - 

[`free_air_anomily = calc_free_air_anomaly(gravity_filtered,latitude,Ellipsoid,orthometric_height)`](https://github.com/Jainam-IITK/CE678_PROJECT/blob/main/calc_free_air_anomaly.m)

Here `Ellipsoid` is a `struct` used for storing important constants of Earth.For example ,You can use it by initiating as - 

`Ellipsoid = make_Ellipsoid('WGS84')` 

### Step 4

From the free-air gravity anomaly, remove the effect of a global gravity field model, a long- wavelength gravity anomaly <img src="https://render.githubusercontent.com/render/math?math=\Delta g_{\mathrm{GGM}}">


<!-- $$
\Delta g_{\mathrm{s} \& \mathrm{mw}}=\Delta g-\Delta g_{\mathrm{GGM}}
$$ --> 

<div align="center"><img src="https://render.githubusercontent.com/render/math?math=%5CDelta%20g_%7B%5Cmathrm%7Bs%7D%20%5C%26%20%5Cmathrm%7Bmw%7D%7D%3D%5CDelta%20g-%5CDelta%20g_%7B%5Cmathrm%7BGGM%7D%7D"></div>
The resultant reduced gravity anomaly  <!-- $\Delta g_{\mathrm{s} \& \mathrm{mw}}$ --> <img src="https://render.githubusercontent.com/render/math?math=%5CDelta%20g_%7B%5Cmathrm%7Bs%7D%20%5C%26%20%5Cmathrm%7Bmw%7D%7D"> contains only the medium and the short wave- lengths.

Download gravity anomaly of any GGM model from [here](http://icgem.gfz-potsdam.de/calcgrid).

TIPS on choosing-

1. Choose latitude and longitude range within the tile of our gravity airborne data we got in 1st step.
2. The website doesent allow resolution less than about 0.01. So make sure choose appropriate resolution so that code does not take long time to execute.

Code- 

1. To import the data

[`del_g_ggm = import_grav_anomaly('filepath',latitude,longitude)`](https://github.com/Jainam-IITK/CE678_PROJECT/blob/main/import_grav_anomaly.m)

[`del_g_s_and_mw = remove_g_ggm(free_air_anomily,del_g_ggm);`](https://github.com/Jainam-IITK/CE678_PROJECT/blob/main/remove_g_ggm.m)

### Step 5

A correction to account for the gravitational attraction of the attraction must be applied
<!-- $$
\Delta g_{\mathrm{s} \ell \mathrm{mw}}^{\mathrm{atm}}=\Delta g_{\mathrm{s} \& \mathrm{mw}}-\delta g_{\mathrm{atm}}
$$ --> 

<div align="center"><img src="https://render.githubusercontent.com/render/math?math=%5CDelta%20g_%7B%5Cmathrm%7Bs%7D%20%5Cell%20%5Cmathrm%7Bmw%7D%7D%5E%7B%5Cmathrm%7Batm%7D%7D%3D%5CDelta%20g_%7B%5Cmathrm%7Bs%7D%20%5C%26%20%5Cmathrm%7Bmw%7D%7D-%5Cdelta%20g_%7B%5Cmathrm%7Batm%7D%7D"></div>

<!-- $$
\begin{aligned}
\delta g_{\mathrm{atm}}=0.871-& 1.0298 \times 10^{-4} H+5.3105 \times 10^{-9} H^{2}-2.1642 \times 10^{-13} H^{3}+9.5246 \times 10^{-18} H^{4}-2.2411 \times 10^{-22} H^{5}
\end{aligned}
$$ --> 

<div align="center"><img src="https://render.githubusercontent.com/render/math?math=%5Cbegin%7Baligned%7D%0A%5Cdelta%20g_%7B%5Cmathrm%7Batm%7D%7D%3D0.871-%26%201.0298%20%5Ctimes%2010%5E%7B-4%7D%20H%2B5.3105%20%5Ctimes%2010%5E%7B-9%7D%20H%5E%7B2%7D-2.1642%20%5Ctimes%2010%5E%7B-13%7D%20H%5E%7B3%7D%2B9.5246%20%5Ctimes%2010%5E%7B-18%7D%20H%5E%7B4%7D-2.2411%20%5Ctimes%2010%5E%7B-22%7D%20H%5E%7B5%7D%0A%5Cend%7Baligned%7D"></div>

Code-

[`g_atm_and_s_mw = remove_g_atm(del_g_s_and_mw,orthometric_height)`](https://github.com/Jainam-IITK/CE678_PROJECT/blob/main/remove_g_atm.m)

### Step 6

Convert the gravity anomaly <!-- $\Delta g_{\mathrm{s} \& \mathrm{mw}}^{\mathrm{atm}}$ --> <img src="https://render.githubusercontent.com/render/math?math=%5CDelta%20g_%7B%5Cmathrm%7Bs%7D%20%5C%26%20%5Cmathrm%7Bmw%7D%7D%5E%7B%5Cmathrm%7Batm%7D%7D"> data points into a grid

Ex Code - 

[`[Latitude_grid,Longitude_grid] = create_grid(latitude,longitude,0.01);`](https://github.com/Jainam-IITK/CE678_PROJECT/blob/main/create_grid.m)

`g_atm_and_s_mw = griddata(latitude,longitude,g_atm_and_s_mw,Latitude_grid,Longitude_grid);`

### Step 7

1. Download the DEM data from [here](http://srtm.csi.cgiar.org/srtmdata/) in ASCII format and of 5'x5' resolution. Remember to use all the tiles that cover up your intereseted area.

2. create a string array of filePaths

   `files = ["file1path","file2path"...]`

Apply gravimetric terrain reduction <!-- $\delta g_{\mathrm{T}}$ --> <img src="https://render.githubusercontent.com/render/math?math=%5Cdelta%20g_%7B%5Cmathrm%7BT%7D%7D"> to  compute the Faye anomaly <!-- $\Delta g_{\text {Faye }}$ --> <img src="https://render.githubusercontent.com/render/math?math=%5CDelta%20g_%7B%5Ctext%20%7BFaye%20%7D%7D">.

Code - 

[`gfaye = calc_gfaye(g_atm_and_s_mw,files,Latitude_grid,Longitude_grid,0.01);`](https://github.com/Jainam-IITK/CE678_PROJECT/blob/main/calc_gfaye.m)

### Step 8

Calculate disturbing potential by using Stokes integral.
<!-- $$
T_{r}=\frac{R}{4 \pi} \iint_{\Omega} \Delta g_{\text {Faye }} S(\psi) d \Omega
$$ --> 

<div align="center"><img src="https://render.githubusercontent.com/render/math?math=T_%7Br%7D%3D%5Cfrac%7BR%7D%7B4%20%5Cpi%7D%20%5Ciint_%7B%5COmega%7D%20%5CDelta%20g_%7B%5Ctext%20%7BFaye%20%7D%7D%20S(%5Cpsi)%20d%20%5COmega"></div>
[`Tr = calc_disturb_potential(Latitude_grid,Longitude_grid,gfaye);`](https://github.com/Jainam-IITK/CE678_PROJECT/blob/main/calc_disturb_potential.m)

### Step 9

By using Bruns’s formula, calculate undulation.
<!-- $$
N_{r}=\frac{T_{r}}{\gamma}
$$ --> 

<div align="center"><img src="https://render.githubusercontent.com/render/math?math=N_%7Br%7D%3D%5Cfrac%7BT_%7Br%7D%7D%7B%5Cgamma%7D"></div>
[`Nr = calc_undulation(Tr,Latitude_grid,Ellipsoid);`](https://github.com/Jainam-IITK/CE678_PROJECT/blob/main/calc_undulation.m)



### Step 10

Restore the undulation <!-- $\left(N_{G G M}\right)$ --> <img src="https://render.githubusercontent.com/render/math?math=%5Cleft(N_%7BG%20G%20M%7D%5Cright)"> corresponding to the removed long-wavelength gravity anomaly

of the GGM model <!-- $\left(\Delta g_{G G M}\right)$ --> <img src="https://render.githubusercontent.com/render/math?math=%5Cleft(%5CDelta%20g_%7BG%20G%20M%7D%5Cright)">   . You will get a co-geoid
<!-- $$
N_{\text {cogeoid }}=N_{r}+N_{\text {GGM }}
$$ --> 

<div align="center"><img src="https://render.githubusercontent.com/render/math?math=N_%7B%5Ctext%20%7Bcogeoid%20%7D%7D%3DN_%7Br%7D%2BN_%7B%5Ctext%20%7BGGM%20%7D%7D"></div>


[`N_co_geoid = calc_n_cogeoid(Nr,nggm,latitude,longitude,Latitude_grid,Longitude_grid);`](https://github.com/Jainam-IITK/CE678_PROJECT/blob/main/calc_n_cogeoid.m)











### PROGRESS

|        Question        |PROGRESS                         |
|----------------|-------------------------------|
|1				 |    Done        |
|2               | Done           |
|3               |Done|
|4               |Done|
|5               |Done|
|6               |Done|
|7               |Done|
|8               |Done|
|9               |Done|
|10              |Done|
|11              |Partially Done|

