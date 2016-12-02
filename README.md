# rlpi package
Robin Freeman, IoZ, Zoological Society of London  
`r format(Sys.Date())`  



##Using the rlpi package 

### Overview

The **rlpi** package calculates indices using the Living Planet Index methodology, as presented in McRae et al. (in review) *The diversity weighted Living Planet Index: controlling for taxonomic bias in a global biodiversity index*.

In summary, indices are calculated using the geometric mean, first combining population trends to the species level, and then across higher taxonomic and geographical groupings. For example, multiple populations within a biogeographic realm will be combined first to generate individual species indices, then combined to taxonomic groups such as *birds*, *mammals*, *reptiles*, *amphibians*, before being combined to an index for the biogeograhic realm

The **rlpi** package works with source data in comma seperated format where each row is composed 
of **popid**, **speciesname**, **year**, **popvalue** (see below). These can be stored be in multiple groups (e.g. a file for Afrotropic bird populations, one for Afrotropical mammal populations, etc), and an 'infile' tells the package where these groups/files are and how to combine them. 

When constructing an index for just a single group, you would need a single data file and a single
infile which points to that data file (see first example below). For multiple groups, the infile would refer to all relevant datafiles and can specify weightings (to allow for weighting by taxonomic, geographical , etc).

The example given below includes an example dataset for terrestrial vertebrates
with a complex infile with multiple weighted groups and a simple infile for Nearctic mammals

NB: At present the code combines population time-series to the species level, generating
an average index for each species, then combines these into higher groups. If you don't wish
to combine to the species level (to generate an index where populations are weighted equally rather then species), then the 'speciesname' column will need to be unique for each population (which can be achieved, for example, by concatenating the speciesname and popid columns)

### Installing the package and examples

First, install the devtools package to enable installing from github:


```r
install.packages("devtools")
```

Then install the **rlpi** package from our github:


```r
library(devtools)
```

```
## Warning: package 'devtools' was built under R version 3.2.5
```

```r
# Install from main ZSL repository online
install_github("Zoological-Society-of-London/rlpi", auth_token = "3e95e9d1c26c0bd8f9fed628b224dbe811064c20")
```

```
## Downloading GitHub repo Zoological-Society-of-London/rlpi@master
## from URL https://api.github.com/repos/Zoological-Society-of-London/rlpi/zipball/master
```

```
## Installing rlpi
```

```
## '/Library/Frameworks/R.framework/Resources/bin/R' --no-site-file  \
##   --no-environ --no-save --no-restore --quiet CMD INSTALL  \
##   '/private/var/folders/w6/_grgw1n52vqgn5l480q1s_m5hs9z1z/T/RtmpKl29d3/devtools1414551501d0/Zoological-Society-of-London-rlpi-2f0ea62c8617def0a43a0f33f250f681915d1851'  \
##   --library='/Library/Frameworks/R.framework/Versions/3.2/Resources/library'  \
##   --install-tests
```

```
## 
```

Then the library can be loaded as normal


```r
# Load library
library(rlpi)
```

And some example data can be extracted from the package:


```r
# Get example data from package
# Copy zipped data to local directory 
file.copy(from=system.file("extdata", "example_data.zip", package = "rlpi"), to=getwd())
# Extract data, this will create a directory of terrestrial LPI data to construct a terrestrial index from.
unzip("example_data.zip")
```

## Example data

Within the example data are a number of 'infiles', these files (take a look at them!) contain links to other files arranged into groups and including weightings. 

For example **terrestrial_class_nearctic_infile.txt** which constructs an index for a single group contains:

```
"FileName"	"Group"	"Weighting"
"class_realms/terrestrial_lpi_rc_p1970_Nearctic_Mammalia.txt"	1	0.175804093567251
```

For now, ignore the 'group' and 'weighting' columns as this is a single group. This references a single 'population' data file (the raw data) in the **class_realms** folder which in this case contains population counts for Neartic mammals (again, have a a look), in the following format:

first six lines of **class_realms/terrestrial_lpi_rc_p1970_Nearctic_Mammalia.txt**:

```
Binomial	ID	year	popvalue
Ovis_canadensis	4618	1950	390
Ovis_canadensis	5328	1950	1500
Myodes_gapperi	4560	1952	17
Sorex_cinereus	4587	1952	3
Napaeozapus_insignis	4588	1952	18
...
```

## Creating an index using example data

Using these files to contruct a Neartic index can be done as follows:


```r
# Make a Neactic LPI 
# Default gives 100 boostraps (this takes a couple of minutes to run on a 2014 Macbook)
Nearc_lpi <- LPIMain("terrestrial_class_nearctic_infile.txt", use_weightings = 1)

# That should have produced a simple plot, but we can use ggplot_lpi to produce a nicer one
ggplot_lpi(Nearc_lpi, ylims=c(0, 2))
```

Similarly, infiles are provided for Nearctic mammals and birds:


```r
# Make a Neactic Mammals LPI 
# Default gives 100 boostraps (this will take a few minutes to run (on a 2014 Macbook))
Nearc_mams_lpi <- LPIMain("terrestrial_mammals_nearctic_infile.txt")

# Nicer plot
ggplot_lpi(Nearc_mams_lpi, ylims=c(0, 2))

# Make a Neactic Mammals LPI 
# Default gives 100 boostraps (this will take a few minutes to run (on a 2014 Macbook))
Nearc_birds_lpi <- LPIMain("terrestrial_birds_nearctic_infile.txt")

# Nicer plot
ggplot_lpi(Nearc_birds_lpi, ylims=c(0, 2))

# We can also combine the two LPIs together in a list
lpis <- list(Nearc_birds_lpi, Nearc_mams_lpi)

# And plot them together 
ggplot_multi_lpi(lpis, xlims=c(1970, 2012), ylims=c(0, 3))

# Can also plot these next to each other, and use some more meaningful titles
ggplot_multi_lpi(lpis, names=c("Birds", "Mammals"), xlims=c(1970, 2012), ylims=c(0, 3), facet=TRUE)
```

## Creating an index using example data (multiple groups and weightings)

This more complex example calculates an index for the terrestrial system, using the input file **terrestrial_class_realms_infile.txt**. This has the following format:

```
"FileName"	"Group"	"Weighting"
"class_realms/terrestrial_lpi_rc_p1970_Afrotropical_Aves.txt"	1	0.260031738834731
"class_realms/terrestrial_lpi_rc_p1970_Afrotropical_Mammalia.txt"	1	0.132963046928134
"class_realms/terrestrial_lpi_rc_p1970_Afrotropical_Herps.txt"	1	0.281115393334845
"class_realms/terrestrial_lpi_rc_p1970_IndoPacific_Aves.txt"	2	0.308085541450115
"class_realms/terrestrial_lpi_rc_p1970_IndoPacific_Mammalia.txt"	2	0.133594615319076
"class_realms/terrestrial_lpi_rc_p1970_IndoPacific_Herps.txt"	2	0.340291386214535
"class_realms/terrestrial_lpi_rc_p1970_Palearctic_Aves.txt"	3	0.295608108108108
"class_realms/terrestrial_lpi_rc_p1970_Palearctic_Mammalia.txt"	3	0.170045045045045
"class_realms/terrestrial_lpi_rc_p1970_Palearctic_Herps.txt"	3	0.218843843843844
"class_realms/terrestrial_lpi_rc_p1970_Neotropical_Aves.txt"	4	0.260026737967914
"class_realms/terrestrial_lpi_rc_p1970_Neotropical_Mammalia.txt"	4	0.0856951871657754
"class_realms/terrestrial_lpi_rc_p1970_Neotropical_Herps.txt"	4	0.326136363636364
"class_realms/terrestrial_lpi_rc_p1970_Nearctic_Aves.txt"	5	0.264985380116959
"class_realms/terrestrial_lpi_rc_p1970_Nearctic_Mammalia.txt"	5	0.175804093567251
"class_realms/terrestrial_lpi_rc_p1970_Nearctic_Herps.txt"	5	0.270102339181287
```

Which refers to 15 different population files, divided into 5 groups (biogeographic realms) using the  "Group" column with different taxonomic gruops within these realms. So group 1 is for the 'Afrotropical' realm and has three population files (Aves, Mammalia and Herps). Weightings are given for these taxonomic groups which specify how much weight each taxonomic group has within the realm index (here using weights relfecting the proportion of species in that taxonomic group in that realm). 


```r
# Whole terrestrial...

# Create a terrestrial index, without using any specified weightings ('use_weightings=0' - so treating taxonomic groups equally at one level, and biogeographic realms equally at the next)
terr_lpi_a <- LPIMain("terrestrial_class_realms_infile.txt", use_weightings=0)

# Run same again, and now weight by class, but equal across realms (see infile for weights)
terr_lpi_b <- LPIMain("terrestrial_class_realms_infile.txt", use_weightings=1)

# Putting the two LPIs together in a list
lpis_comp <- list(terr_lpi_a, terr_lpi_b)

# And plotting them together 
ggplot_multi_lpi(lpis_comp, xlims=c(1970, 2012))
ggplot_multi_lpi(lpis_comp, xlims=c(1970, 2012), names=c("Unweighted", "Weighted"), facet=TRUE)
```

We can also compare the impact of removing particular group from the indices - for example, removing groups with low representaton (figure X in the paper).

```

```

### See Also

Some functions are also provided for creating infiles from tabular data: [Creating Infiles](creating_infiles.html)
