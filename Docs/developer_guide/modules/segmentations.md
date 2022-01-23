# Segmentations

## Segmentation labelmap file format (.seg.nrrd)

Segmentation is stored in a standard NRRD image file with custom fields to store additional metadata.

The **[slicerio](https://pypi.org/project/slicerio/) Python package** can read .seg.nrrd files and import segments into numpy arrays and dictionary objects. The package can be used in any Python environment (installed using `pip install slicerio`) - not just in 3D Slicer.

### Image data

If segments do not overlap then the file has 3 spatial dimensions. Even if a single-slice image is segmented, the spatial dimension must be still 3 because that is required for specification origin, spacing, and axis directions in 3D space. If segments overlap then the file has one `list` dimension and 3 spatial dimensions. Each 3D volume in the list is referred to as a `layer`.

### Metadata

Additional metadata is stored in custom data fields (starting with `Segmentation_` or `SegmentN_` prefixes), which provide hints on how the segments should be displayed or what they contain.

#### Common custom fields

- `Segmentation_ContainedRepresentationNames`: names of segmentation representations (separated by `|` character) that should be displayed. Common representation names include `Binary labelmap`, `Closed surface`, `Planar contours`, `Fractional labelmap`.
- `Segmentation_ConversionParameters`: parameters for algorithms that compute requested representations. Each parameter definition is separated by the `&` character. A parameter definition consists of parameter name, value, and description text, separated by `|` character.
- `Segmentation_MasterRepresentation`: defines what representation is stored in the file. It is most commonly `Binary labelmap`, but there are other representations, too, which are stored in an image volume - such as `Fractional labelmap`.
- `Segmentation_ReferenceImageExtentOffset`: This field allows storing an image with arbitrary extent in a NRRD file. NRRD file only stores size of the volume. This field stores voxel coordinates (separated by spaces), therefore the extent can be computed as `[ReferenceImageExtentOffsetI, ReferenceImageExtentOffsetI+sizeI-1, ReferenceImageExtentOffsetJ, ReferenceImageExtentOffsetJ+sizeJ-1, ReferenceImageExtentOffsetK, ReferenceImageExtentOffsetk+sizeK-1]`.

#### Segment custom fields

These fields specify properties for each segment. `N` refers to the segment index, which is an integer between 0 and `NumberOfSegments - 1`.

- `SegmentN_ID`: identifier of the segment, it is unique within the segmentation. Can be used for unanmiguously refer to a segment even if the segment's display name is changed.
- `SegmentN_Name`: display name of the segment (the name that is shown to users).
- `SegmentN_NameAutoGenerated`: if value is `0` then it means that `SegmentN_Name` was chosen manually by the user, if value is `1` then it means that the segment's name is generated automatically (for example, determined from terminology).
- `SegmentN_Color`: recommended segment display color, defined with red, green, blue values (between 0.0 - 1.0) separated by space character.
- `SegmentN_ColorAutoGenerated`: if value is `0` then it means that `SegmentN_Color` was chosen manually by the user, if value is `1` then it means that the segment's color is generated automatically (for example, color defined based on terminology).
- `SegmentN_Extent`: 6 space-separated values [`minI`, `maxI`, `minJ`, `maxJ`, `minZ`, `maxZ`] defining extent of non-empty region within the segment.
- `SegmentN_Tags`: List of key-value pairs for storing additional information about the segment. Key and value is separated by the `:` character,
pairs are separated from each other by the `|` character.
- `SegmentN_Layer`: Index of the 3D volume that contains the segment. Its value is between 0 and `DimensionAlongTheListImageAxis - 1`. For segmentations in which the segments do not overlap, the segmentation can be represented as a single 3D volume with 1 layer. Upon saving, segments are typically collapsed to as few layers as possible.
- `SegmentN_LabelValue`: The scalar value used to represent the segment within its own layer. Segments in separate layers can have the same label value.

##### TerminologyEntry tag

 A frequently used key is `TerminologyEntry`, which defines what the segment contains using DICOM compliant terminology. Value stores 7 parts: `terminology context name`, `category`, `type`, `type modifier`, `anatomic context name`, `anatomic region`, and `anatomic region modifier`. Parts are separated from each other by `~` character. Five of these parts - `category`, `type`, `type modifier`, `anatomic region`, and `anatomic region modifier` are defined by a triplet of (`coding scheme designator`, `code value`, and `code meaning`). Components of the triplet are separated by `^` character.

Example:

A mass in the right side of the adrenal gland would be encoded with the following the tag:

```
TerminologyEntry:
Segmentation category and type - 3D Slicer General Anatomy list
~SCT^49755003^Morphologically Altered Structure
~SCT^4147007^Mass
~^^
~Anatomic codes - DICOM master list
~SCT^23451007^Adrenal gland
~SCT^24028007^Right"
```

Interpretation:

- `terminology context name`: `Segmentation category and type - 3D Slicer General Anatomy list`
- `category`: `codingScheme`=`SCT` (Snomed Clinical Terms), `codeValue`=`49755003`, `codeMeaning`=`Morphologically Altered Structure`
- `type`: `codingScheme`=`SCT`, `codeValue`=`4147007`, `codeMeaning`=`Mass`
- `type modifier`: not specified
- `anatomic context name`: `Anatomic codes - DICOM master list`
- `anatomic region`: `codingScheme`=`SCT`, `codeValue`=`23451007`, `codeMeaning`=`Adrenal gland`
- `anatomic region modifier`: `codingScheme`=`SCT`, `codeValue`=`24028007`, `codeMeaning`=`Right`


## Segmentation surface file format (.seg.vtm)

Segmentation is stored in a standard VTK multiblock data set file format. Multiblock data set consists of an index (.vtm) file that stores the file path of each block. Each block file is a VTK polydata (.vtp) file, typically stored in a subdirectory.

### Metadata

Metadata fields are stored in the polydata files (.vtp) as arrays in `FieldData`:

- `Segmentation_MasterRepresentation`: type: `String`, definition: see in "Common custom fields" section above
- `Segmentation_ConversionParameters`: type: `String`, definition: see in "Common custom fields" section above
- `Segmentation_ContainedRepresentationNames`: type: `String`, definition: see in "Common custom fields" section above
- `Segment_ID`: type = `String`, definition: see in "Segment custom fields" section above
- `Segment_Name`: type = `String`, definition: see in "Segment custom fields" section above
- `Segment_NameAutoGenerated`: type = `String`, definition: see in "Segment custom fields" section above
- `Segment_Color`: type = `Float64`, number of components = 3, definition: see in "Segment custom fields" section above
- `Segment_ColorAutoGenerated`: type = `String`, definition: see in "Segment custom fields" section above
- `Segment_Tags`: type = `String`, definition: see in "Segment custom fields" section above