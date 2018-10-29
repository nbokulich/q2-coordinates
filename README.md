# q2-coordinates
A qiime2 plugin supporting methods for geographic mapping of qiime2 artifact data or metadata.

q2-coordinates makes it easy to plot geographic coordinates and associated (meta)data on beautiful topographic and street maps.

Map tiling, resolution calculation, and coordinate projection occur automatically. All the user needs to do is input a two-dimensional list of geocoordinates to plot, in decimal degrees, as shown in the examples below.

Currently, StamenTerrain, Open Street Maps, and Google Maps are supported, producing high-quality maps from anywhere on planet Earth.

Map projections are quick for small maps, but may take several minutes for very large maps (e.g., trans-continental).

# Examples
In the examples below we will use some bacterial 16S rRNA gene amplicon sequence data collected from Californian vineyards, as described by [Bokulich et al. 2016](https://doi.org/10.1128/mBio.00631-16). 😎🍷

## Plotting geocoordinates colored by alpha diversity values
This visualizer takes a SampleData[AlphaDiversity] artifact and sample metadata TSV as input, and plots sample coordinates on the built-in maps. Sample points are colored as a function of alpha diversity values.
```
cd q2-coordinates/q2_coordinates/tests/data/

qiime diversity alpha \
    --i-table even_table.qza \
    --p-metric observed_otus \
    --o-alpha-diversity alpha_diversity1

qiime coordinates draw-map \
    --m-metadata-file alpha_diversity.qza \
    --m-metadata-file chardonnay.map.txt \
    --p-latitude latitude \
    --p-longitude longitude \
    --p-column observed_otus \
    --o-visualization diversity-map
```

![Alt text](./examples/alpha-diversity.jpg?raw=true "coordinates colored by observed species")

Note that _any_ metadata-transformable artifacts can be used as a `metadata-file` input to this command, so this opens the door to many other data types, e.g., PCoA results, predictions, etc.

HINT: If you are not sure what the `column` name is for your artifact of interest, use `qiime metadata tabulate` to view as a dataframe.

## Plotting geocoordinates colored by metadata category values
We can use the same visualizer action for plotting alpha diversity values to color sample points as a function of continuous or categorical sample metadata. To do this, we simply add the "category" parameter to use that category from the sample metadata instead of alpha diversity values (which must still be input for the time being, until qiime2 permits optional artifacts). The plot below shows the various vineyard sites (indicated by anonymous numbers) where samples were collected.
```
qiime coordinates draw-map \
    --m-metadata-file chardonnay.map.txt \
    --p-column vineyard \
    --p-latitude latitude \
    --p-longitude longitude \
    --p-discrete \
    --o-visualization vineyard-map
```
![Alt text](./examples/vineyard-map.jpg?raw=true "coordinates colored by metadata values")

## Converting (geo)coordinates to a distance matrix
This is a cinch with q2-coordinates! To calculate geodesic distance from geocoordinate data, use the `geodesic-distance` method:
```
qiime coordinates geodesic-distance \
    --m-metadata-file chardonnay.map.txt \
    --p-latitude latitude \
    --p-longitude longitude \
    --o-distance-matrix dm.qza
```

This computes geodesic distance (in meters) between each point. Note that samples with missing values are ignored.

We can also construct a distance matrix from 2D or 3D spatial coordinates using the `euclidean-distance` method:
```
qiime coordinates euclidean-distance \
    --m-metadata-file xyz-coordinates.tsv \
    --p-x x \
    --p-y y \
    --p-z z \
    --o-distance-matrix xyz-dm.qza
```

We can use these distance matrices for other useful QIIME 2 methods, e.g., to compute a mantel test comparing two different distance matrices. For example, we can compare Bray-Curtis dissimilarities between microbial communities to geospatial distances between our vineyard samples:
```
qiime diversity mantel \
    --i-dm1 dm.qza \
    --i-dm2 distance.qza \
    --p-intersect-ids \
    --o-visualization mantel.qzv
```
