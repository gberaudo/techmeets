# Graphhopper with Swissalti3d open data

## Grant goal

Graphhopper is a 3D-enabled Opensource routing engine:

- it returns 3d geometries of the routed path;
- it uses elevation in its routing profile, for example to avoid hills.

Unfortunately, it only supports a few approximative DEMs by default, making its results not as powerful as it could.

My goal was to implement an elevation provider for the very precise SwissAlti3d DEM.

## Retrieving the DEM tiles

The DEM is available as small tiles, through simple HTTP requests.
There is a difficulty though.

```java
  /**
     * Fetch some remote files and create a mapping from (x,y) to tile URLs. This is necessary because the URL naming
     * also depends on the year the file data has been captured. See the technical specification of the dataset for details.
     * @param downloader
     * @return the URL mapping
     * @throws IOException remote file retrieving errors
     */
    private Map<String, String> fetchTileMapping(Downloader downloader) throws IOException {
        File cachedMappingFile = new File(cacheDir, "mappings.csv");
        if (!cachedMappingFile.exists()) {
            if (!cacheDir.exists()) {
                cacheDir.mkdirs();
            }
            String response = downloader.downloadAsString(Swissalti3dElevationProvider.tilingSchemeUrl, false);
            // ex: {"href":"https://ogd.swisstopo.admin.ch/resources/ch.swisstopo.swissalti3d-9u0iezRG.csv"}
            String csvUrl = response.split("\"")[3];
            downloader.downloadFile(csvUrl, cachedMappingFile.getAbsolutePath());
        }

        String csv = FileUtils.readFileToString(cachedMappingFile, "ascii");
        Scanner scanner = new Scanner(csv);
        Map<String, String> mapping = new HashMap<>();
        while (scanner.hasNextLine()) {
            // ex: https://data.geo.admin.ch/ch.swisstopo.swissalti3d/swissalti3d_2019_2501-1120/swissalti3d_2019_2501-1120_2_2056_5728.tif
            String line = scanner.nextLine();
            int idx = line.lastIndexOf("/");
            if (idx >= 0) {
                String part = line.substring(idx);
                String[] split = part.split("_");
                String xy = split[2];
                mapping.put(xy, line);
            }
        }
        return mapping;
    }
```

## Caching the rasters - locality

We wants to add 3D to all coordinates found in the OpenStreetMap OSM file.
For each (x,y) we locate the raster tile containing the elevation data.
This is reasonable because the OSM file is not iterated randomly (geographically speaking), but in a way that preserves "locality".

```java
    private Raster getRasterContainingCoordinate(int x, int y) throws  IOException {
        // Best case: return it from the cache
        String xy = xyToTileKey(x, y);

        Raster raster = rasterByXY.get(xy);
        if (raster != null) {
            return raster;
        }

        // lazy load the map of tile URLs
        if (tileUrlByXY == null) {
            tileUrlByXY = this.fetchTileMapping(downloader);
        }

        // Retrieve remote tile if not existing locally
        String tileUrl = tileUrlByXY.get(xy);
        File cachedFile = getCachedTileFile(xy, tileUrl);
        if (!cachedFile.exists()) {
            if (!cacheDir.exists()) {
                cacheDir.mkdirs();
            }
            downloader.downloadFile(tileUrl, cachedFile.getAbsolutePath());
        }

        // Parse tif file as raster and return it
        raster = readTifFile(cachedFile);
        if (rasterByXY.size() > 100) {
            logger.debug("Clearing in memory raster cache");
            rasterByXY.clear();
        }
        rasterByXY.put(xy, raster);
        return raster;
    }
```

## Reading tif data

I tried many ways to read the tif data: it all failed because GeoTiffs comes in several versions that are not all well supported by the geotiff libraries. The solution was to rely on a recent JDK...

```java

    Raster readTifFile(File tifFile) {
        SeekableStream ss = null;
        try {
            InputStream is = new FileInputStream(tifFile);
            Raster raster = ImageIO.read(is).getData();
            return raster;
        } catch (Exception e) {
            e.printStackTrace();
            throw new RuntimeException("Can't decode " + tifFile.getName(), e);
        } finally {
            if (ss != null)
                Helper.close(ss);
        }
    }
```

## Conclusions

I implemented "on-the-fly" download and use of geotiff DEM data to do precise 3d routing in Graphhopper.
It is a reasonably simple solution, that _does_ actually work.
I only had time to try on a small POC example but I am confident this approach can be generalized:

- to a large area;
- to other DEM providers: IGN? Volunteers?

I would like to thank Camptocamp to allowing me to experiment with this and to the Graphhopper maintainers for their precious advices.


## Downloads

*Graphhopper* is an Opensource (Apache v2) routing engine for OpenStreetMap.
It is written in Java and available from https://github.com/graphhopper/graphhopper.
My fork and work is in https://github.com/gberaudo/graphhopper/tree/add_swissalti3d_elevation_provider

*Swissalti3d" is an extremely precise Digital Elevation Model (DEM) published by the Federal Office of Topography (Swisstopo).
It is available as opendata from https://www.swisstopo.admin.ch/en/height-model-swissalti3d.
