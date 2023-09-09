# EMR PYSPARK PROJECT
## Project 1

## Table of contents

- [About](#about)
- [Solution](#solution)
- [Results](#results)

## About
Our task was to implement the solution that: \
Analyze the data containing information on the variation in elevation by selecting groups of areas with the highest growth (continent Europe). The height increase in a given location should be measured from at least 10 measurement points. Determine 5 groups of areas in relation to the average value of height increase. Please plot the detected areas on the map. 

We decided to use AWS EMR service and Spark framework for proccessing big data to get know working with clusters and cloud enviroment. Thanks to AWS Learner Lab we were able to use EMR service for free.

Dataset:  https://registry.opendata.aws/terrain-tiles/ \
AWS EMR: https://aws.amazon.com/emr/ \
PySpark: https://spark.apache.org/docs/latest/api/python/index.html 

## Solution
Flow: <br>
  1) Define Geographical Bounds: Set the geographical coordinates for Europe as a tuple of (max_latitude, min_longitude, min_latitude, max_longitude).

  2) Generate Tile URLs: Generate the URLs for map tiles that span the defined geographical area using the function generateTileUrls.

  3) Load Image Data: Load the image data (map tiles) from the generated URLs into a Spark DataFrame using PySpark's read.format().

  4) Refine Image Data: Extract the x and y tile indices from the origin column of the DataFrame and sort the rows based on these indices.

  5) Extract Pixel Data: Select only the data column from the DataFrame, which contains the pixel information.

  6) Reshape Pixel Data: Convert the PySpark DataFrame to an RDD and reshape the pixel data into NumPy arrays of shape (256, 256, 3).

  7) Calculate Elevation: Use the function calculateElevation to transform the pixel data to elevation data.

  8) Compute Gradients: Compute the x and y gradients using Sobel filters via OpenCV's cv2.Sobel() function.

  9) Compute Gradient Magnitude: Calculate the magnitude of the gradients from the x and y components.

  10) Generate Thresholds: Divide the gradient magnitude data into different groups using the function generateThresholds.

  11) Apply Color Mapping: Apply color mapping to the gradient magnitude based on the threshold groups using the function applyColorMapping.

  12) Display and Save Output: Use the function displayCompositeMap to display the final composite map. The composite map is also saved as a PNG file.

Using JupyterNotebook we were able to divide each function to each cell to achieve clearness of code. Each function before implementation has a broad description how it works. \
Also here, we listing each function with explanation:
- [Functions](#functions)
  - [latLonToTileCoord](#latlontotilecoord)
  - [tileCoordToLatLon](#tilecoordtolatlon)
  - [generateTileUrls](#generatetileurls)
  - [displayImage](#displayimage)
  - [displayCompositeMap](#displaycompositemap)
  - [calculateElevation](#calculateelevation)
  - [generateThresholds](#generatethresholds)
  - [applyThresholds](#applythresholds)
  - [applyColorMapping](#applycolormapping)
  - [extractXCoordinate](#extractxcoordinate)
  - [extractYCoordinate](#extractycoordinate)
  - [runPipeline](#runpipeline)

  ## Functions

  ## latLonToTileCoord
  ### Description:

  This function converts geographical coordinates (latitude and longitude) to tile coordinates for a given zoom level. Tile coordinates are essential in map rendering to identify which portions of the map to display.
  ### Parameters:

    latitude: The latitude coordinate of the location, expected to be a float value ranging from -90 to 90.
    longitude: The longitude coordinate of the location, expected to be a float value ranging from -180 to 180.
    zoom_level: The zoom level for the map, determining the resolution of the tile. Higher zoom levels correspond    
    to higher detail.

  ### Returns:

  The function returns a tuple (x_tile, y_tile) that contains the x and y coordinates of the tile.

  ## tileCoordToLatLon
  ### Description:

  This function takes tile coordinates (x_coord, y_coord) and a zoom level (zoom_level) as input arguments. It returns the corresponding geographical coordinates (latitude, longitude) based on the provided tile coordinates and zoom level.
  ### Parameters:

    x_coord: The x-coordinate of the tile.
    y_coord: The y-coordinate of the tile.
    zoom_level: The zoom level at which the tile exists.

  ### Returns:

  A tuple containing the geographical coordinates (latitude, longitude).
  ### Working Principle:

    It first calculates a divisor as 2zoom_level2zoom_level.
    The longitude is calculated using the equation x_coorddivisor×360.0−180.0divisorx_coord​×360.0−180.0.
    The latitude is calculated by:
        Taking the inverse hyperbolic sine of π×(1−2×y_coorddivisor)π×(1−divisor2×y_coord​).
        Taking the arctangent of the result to get the latitude in radians.
        Converting the latitude from radians to degrees.

  ## generateTileUrls
  ### Description:

  This function takes in a zoom level and two sets of geographical coordinates (latitude1, longitude1) and (latitude2, longitude2). It returns a list of URLs from which tiles can be retrieved. The list contains tiles that cover the area between     the provided geographical coordinates at the given zoom level. The function also returns the dimensions of the tile grid that covers the region (number of tiles along the x and y axes).
  ### Parameters:

    zoom_level: The zoom level for which you want to retrieve tiles.
    latitude1, longitude1: The latitude and longitude of the first corner of the bounding box.
    latitude2, longitude2: The latitude and longitude of the second corner of the bounding box.

  ### Returns:

    A list of URLs for tile retrieval.
    The number of tiles in the x-direction.
    The number of tiles in the y-direction.

  ### Working Principle:

    Determine the minimum and maximum latitude and longitude to form a geographical bounding box.
    Convert these geographic coordinates to tile coordinates using latLonToTileCoord.
    Generate a list of tile URLs covering the area within the tile-space bounding box

  ## displayImage
  ### Description:

  This function takes an array representing image data and displays it as a grayscale image using the Matplotlib library. It clears any existing plots before rendering the new image.
  ### Parameters:

    image_data: A NumPy array containing the image data. The shape and dimensions of the array should be compatible with what matplotlib.pyplot.imshow can handle. For example, it can be a 2D array for grayscale images or a 3D array for color         images.

    ### Returns:

    No return value. The function displays the image using Matplotlib's imshow method and sets the colormap to grayscale.

  ### Working Principle:

    Clears the current figure using plt.clf().
    Sets the colormap to grayscale using plt.set_cmap('gray').
    Displays the image using plt.imshow()

  ## displayCompositeMap
  ### Description:

  This function takes a set of image tiles, as well as the number of tiles in the x and y directions, and then constructs and displays a composite map using the Matplotlib library.  
  ### Parameters:

    tile_set: A list of NumPy arrays, each representing a tile. The list should be in a form that allows for concatenation into a full map.
    x_count: The number of tiles along the x-axis.
    y_count: The number of tiles along the y-axis.

  ### Returns:

    No return value. The function constructs and displays the composite map using Matplotlib's imshow method.

  ### Working Principle:

    For each row of tiles along the x-axis, the function concatenates the corresponding tiles along the y-axis. This produces a list of "tile columns."
    These tile columns are then concatenated along the x-axis to create a final composite map.
    The Matplotlib library is then used to display this composite map.

  ## calculateElevation
  ### Description:

  This function takes a NumPy array containing pixel data for a tile image and calculates the corresponding elevation levels. The pixel data is assumed to be in the format of 3 separate color channels: Red, Green, and Blue.
  ### Parameters:

    pixel_data: A NumPy array containing the pixel data for a single image tile.

  ### Returns:

    A NumPy array of the same shape as pixel_data, but containing calculated elevation levels instead of color information.

  ### Working Principle:

    The function splits the pixel_data NumPy array into its Red, Green, and Blue channels using OpenCV's cv2.split method.
    It then calculates the elevation according to the formula: Elevation=(Red×256.0+Green+Blue256.0)−32768.0Elevation=(Red×256.0+Green+256.0Blue​)−32768.0
    Any negative elevation values are replaced with zero, as negative values are considered to be invalid.
  
  ## generateThresholds
  ### Description:

  This function takes a list of NumPy arrays representing pixel data and calculates threshold values used to divide the elevation data into different groups. The function also prints the threshold values associated with each percentage group.     The pixel data is assumed to be grayscale.
  ### Parameters:

    pixel_data: A list of NumPy arrays containing the pixel data for multiple image tiles.

  ### Returns:

    A list of threshold values that will be used to divide the elevation data into different groups.
  
  ### Working Principle:

    The function parallelizes the pixel_data using Spark's RDD.
    It calculates the average value of each array using OpenCV's cv2.mean and sorts them in ascending order.
    The function calculates threshold values at specific percentages (95%, 80%, 60%, 45%, and 30%) of the sorted  mean values and prints them.
    These threshold values are then returned for further use in data processing.

   ## applyThresholds
   ### Description:

    This function applies predefined threshold values to an input NumPy array, effectively dividing the data into different groups based on these thresholds. It creates a new array where the elements are replaced with integers from 0 to 5,           representing which threshold range each original value falls into.
   ### Parameters:

    input_array: A NumPy array that contains the data to which you want to apply the thresholds.
    hresholds: A list of numerical values used for defining threshold ranges.

   ### Returns:

    A NumPy array with elements replaced by integers (from 0 to 5), indicating which threshold range each original value belongs to.

  ### Working Principle:

    The function takes a copy of the input array.
        It then applies a series of NumPy where operations to replace the values in the array based on which threshold range they fall into:
        Values greater than or equal to thresholds[0] become 0 \
        Values between thresholds[0] and thresholds[1] become 1 \ 
        Values between thresholds[1] and thresholds[2] become 2 \ 
        Values between thresholds[2] and thresholds[3] become 3 \
        Values between thresholds[3] and thresholds[4] become 4 \
        Values less than thresholds[4] become 5

    ## applyColorMapping
    ### Description:

    This function takes a set of image pixel data and a list of threshold values, applies the threshold ranges to each pixel's value, and returns a new set of data with the pixel values converted to integers representing different color mappings.
    ### Parameters:

    pixel_data: A set of image pixel data, usually in the form of a list of arrays.
    thresholds: A list of numerical values used for defining threshold ranges.

    ### Returns:

    A new set of image pixel data where each pixel value has been mapped to an integer representing a color map value.

    ### Working Principle:

    The function uses Spark's parallelize method to distribute the given pixel_data across multiple nodes for parallel processing.
    It applies the map transformation to each element (usually an array representing a tile or a portion of an image) in the distributed data. The mapping function used is applyThresholds, which will       change each pixel's value based on the            provided thresholds.
    The transformed data is then collected back into a list using Spark's collect method.
  
    ## extractXCoordinate

    ### Description:

  This function takes a file path string and extracts the x-coordinate value based on the path structure. This is useful when the file paths are structured in a way that they contain spatial              coordinates (e.g., tile indexes for map tiles).
    ### Parameters:

    filepath: A string containing the file path from which the x-coordinate will be extracted.

    ### Returns:

    An integer representing the extracted x-coordinate.

    ### Working Principle:

    The function splits the filepath string using the slash / as a delimiter.
    It then accesses the 6th index (assuming 0-based indexing) of the resulting list to obtain the x-coordinate, which is converted to an integer.

    ## extractYCoordinate
    ### Description:

    This function takes a file path string and extracts the y-coordinate value based on the path structure. This is useful when the file paths are structured in a way that they contain spatial              coordinates (e.g., tile indexes for map tiles).
    ### Parameters:

    filepath: A string containing the file path from which the y-coordinate will be extracted.

    ### Returns:

    An integer representing the extracted y-coordinate.

    ### Working Principle:

    The function splits the filepath string using the slash / as a delimiter.
    It then accesses the 6th index (assuming 0-based indexing) of the resulting list to obtain the y-coordinate.
    The y-coordinate part may contain the file extension, so it's further split using the dot . as a delimiter, and the first part is converted to an integer

    ## runPipeline

    ### Description:

  This function serves as the main driver of the elevation data processing and visualization pipeline. It orchestrates the sequence of operations required to transform tile data into a color-mapped,       composite representation of elevation             gradients across a given geographical area (in this case, Europe).

  ### Working Steps:

  Define Geographical Bounds: Set the geographical coordinates for Europe as a tuple of (max_latitude, min_longitude, min_latitude, max_longitude).

  Generate Tile URLs: Generate the URLs for map tiles that span the defined geographical area using the function generateTileUrls.

  Load Image Data: Load the image data (map tiles) from the generated URLs into a Spark DataFrame using PySpark's read.format().

  Refine Image Data: Extract the x and y tile indices from the origin column of the DataFrame and sort the rows based on these indices.

  Extract Pixel Data: Select only the data column from the DataFrame, which contains the pixel information.

  Reshape Pixel Data: Convert the PySpark DataFrame to an RDD and reshape the pixel data into NumPy arrays of shape (256, 256, 3).

  Calculate Elevation: Use the function calculateElevation to transform the pixel data to elevation data.

  Compute Gradients: Compute the x and y gradients using Sobel filters via OpenCV's cv2.Sobel() function.

  Compute Gradient Magnitude: Calculate the magnitude of the gradients from the x and y components.

  Generate Thresholds: Divide the gradient magnitude data into different groups using the function generateThresholds.

  Apply Color Mapping: Apply color mapping to the gradient magnitude based on the threshold groups using the function applyColorMapping.

  Display and Save Output: Use the function displayCompositeMap to display the final composite map. The composite map is also saved as a PNG file.

  ### Outputs:

  Displays the composite map showing elevation gradients with applied color mapping.
      Saves the composite map as a PNG file named color_mapped_output.png

## Results

We achieved the map with marked regions in Europe:
![Europe](europe.png)
