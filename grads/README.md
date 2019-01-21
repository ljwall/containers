GrADS Docker Container
===

Builds GrADS version 2.1.1.b0 in an Alpine based docker container

## Building the Dockerfile
To build the dockerfile run
```
docker build -t grads .
```
in the folder with the file.

## Running GrADS
First we need some example data, grab your favorite grib2 output from a NWP model. If you don't have one handy, you can download one from NOMADS for GFS at [ftp://ftpprd.ncep.noaa.gov/pub/data/nccf/com/gfs/prod/](ftp://ftpprd.ncep.noaa.gov/pub/data/nccf/com/gfs/prod/)

Build the dockerfile using the step above, or pull from the Quay.io registry
```
docker pull quay.io/occ_data/grads
```

Run the container mounting the directory with your grib file to /data

```
docker run -v $(pwd):/data --name grads -d --rm -ti grads
```

Also, providing X11 access:

```
xhost +local: // To allow permision for X11 connection
docker run -v $(pwd):/data -e DISPLAY=$DISPLAY -v /tmp/.X11-unix:/tmp/.X11-unix  --name grads --rm -ti grads
```

Maybe run this after:

```
xhost -local:
```

See the [referece and tutorial](http://cola.gmu.edu/grads/gadoc/gadoc.php).

to make the required `ctl` and `idx` files from a grib file run the following in the container:

```
g2ctl filename.grb > filename.ctl
gribmap -e -i filename.ctl  // Generates gribmap file filename.grb.idx
```

Finally here is an example of how to use GrADS to generate a geotiff file of the 2-m temperature. This example uses a GFS grib2 file called gfs.t12z.pgrb2.0p25.f003.
```
# Generate the grib control file
docker exec -ti grads g2ctl gfs.t12z.pgrb2.0p25.f003 >gfs.t12z.pgrb2.0p25.f003.ctl

# Generate the grib index file
docker exec -ti grads gribmap -i gfs.t12z.pgrb2.0p25.f003.ctl

# Run GrADS
docker exec -ti grads grads -lb
>open fs.t12z.pgrb2.0p25.f003.ctl
>set gxout geotiff
>d tmp2m
>quit
```

This should have saved a geotiff called gradsgeo.tif which you can then plot with your favorite GIS software to produce something like this.
![Example 2-m Temperature Output](/grads/2m-temp.png?raw=true "Example 2-m Temperature Output")
