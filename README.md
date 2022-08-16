# Gradient-forest-analysis-
Gradient forest for Queensland fruit fly based on input files generated by Elahe Parvizi for the Invasomics project, based on the analysis undertaken by cao et al (2021) in the following paper entitled:

"Local climate adaptation and gene flow in the native range of two co-occurring fruit moths with contrasting invasiveness"

https://onlinelibrary.wiley.com/doi/full/10.1111/mec.16055.


R script this analysis was based off was found in the Dryad repository for the above paper https://datadryad.org/stash/dataset/doi:10.5061/dryad.m0cfxpp20 with changes an clarifications for further dependecies and specifics for this analysis outlined below.

## External software install- gradientforest

install.packages("gradientForest", repos="http://R-Forge.R-project.org")

install.packages("extendedForest", repos="http://R-Forge.R-project.org")

## Libraries installed and used


`library(tidyr)`

`library(sp)`

`require(raster)`

`require(rgdal)`

`library(tidyr)`

`library(sp)`


## Generating of input files

SNP frequency files and climate data for each sample provided by Elahe Parvizi.

World climate data generated from bioclim data https://www.worldclim.org/data/worldclim21.html.


Files generated in r for clim. list, making list of paths for each layer

`clim.list <- dir("./Climate_Data/", full.names=T, pattern='.tif') `

Then stacked into a single object:

` clim.layer <-  stack(clim.list) `


`clim.points <- extract(clim.layer.crop, sample.coord.sp) `

`clim.points <- cbind(sample.coord, clim.points)  `

`write.table(clim.points, "clim.points", sep="\t", quote=F, row.names=F)  `


## Altering previously defined scripts for filtering to this analysis

env.gf variable was filtered based on climate/ variable corerlation analysis between worldclim and sampling locations. this narrowed down to 6 bioclimactic variables:

`env.gf <- cbind(clim.points[ , c("bio_3", "bio_5", "bio_8", "bio_9", "bio_12", "bio_19") ], pcnm.keep) `

## Spatial analysis based on the gf

crop climate data layers to just the area included in the analysis

`extent <- c(-150, 167, 5, 22) `

`clim.layer.crop <- crop(clim.layer, extent)`

Plot maps of cropped layers

`pdf("clim.layer.crop.pdf")`

`plot(clim.layer.crop)`

`dev.off()`

###Spatial mapping- if have tidyr loaded unload as the extract function masks that from raster

`clim.land <- extract(clim.layer.crop, 1:ncell(clim.layer.crop), df = TRUE)

`clim.land <- na.omit(clim.land)

`clim.land_subset=subset(clim.land, select = -c(wc2.1_10m_bio_1, wc2.1_10m_bio_2, wc2.1_10m_bio_4, wc2.1_10m_bio_6, wc2.1_10m_bio_7, wc2.1_10m_bio_10, wc2.1_10m_bio_11, wc2.1_10m_bio_13, wc2.1_10m_bio_14, wc2.1_10m_bio_15, wc2.1_10m_bio_16, wc2.1_10m_bio_17, wc2.1_10m_bio_18))`

`colnames(clim.land_subset) <- c("ID", "bio_12", "bio_19", "bio_13", "bio_15"," bio_8", "bio_9")`

pred <- predict(gf, clim.land_subset[,-1])  #note the removal of the cell ID column with [,-1]

###Mapping

pca <- prcomp(pred, center=T, scale.=F)

#Assign PCs to colors
r <- pca$x[, 1]
g <- pca$x[, 2]
b <- pca$x[, 3]

#Scale colors
r <- (r - min(r))/(max(r) - min(r)) * 255
g <- (g - min(g))/(max(g) - min(g)) * 255
b <- (b - min(b))/(max(b) - min(b)) * 255

#Define raster properties with existing one
mask <- clim.layer.crop$wc2.1_10m_bio_12

#Assign color to raster
rastR <- rastG <- rastB <- mask
rastR[clim.land_subset$ID] <- r
rastG[clim.land_subset$ID] <- g
rastB[clim.land_subset$ID] <- b

#Stack color rasters
rgb.rast <- stack(rastR, rastG, rastB)

pdf("GF_Map.pdf")
plotRGB(rgb.rast, bgalpha=0)
points(clim.points$Longitude, clim.points$Latitude)
dev.off()
