# VK_EXT_hdr_SMPTE_ST_2094_50_metadata (3)

## Name

VK_EXT_hdr_SMPTE_ST_2094_50_metadata - device extension

## VK_EXT_hdr_SMPTE_ST_2094_50_metadata

### Name string

VK_EXT_hdr_SMPTE_ST_2094_50_metadata

### Extension Type

Device extension

### Registered Extension Number

XXX

### Revision

XXX

### Ratification Status

Not ratified

### Extension and Version Dependencies

[VK_EXT_hdr_metadata](https://docs.vulkan.org/refpages/latest/refpages/source/VK_EXT_hdr_metadata.html)

### Contact

* [Christopher Cameron](ccameron@chromium.org)

## Other Extension Metadata

### Last Modified Date

2026-05-06

### IP Status

XXX

### Contributors

* Alec Mouri (Google)
* Christopher Cameron (Chromium)
* John Reck (Google)

## Description

This extension provides a mechanism for an application to attach
SMPTE ST 2094-50 metadata to a swapchain via the
`VkHdrSmpteSt_2094_50_EXT` structure.

The SMPTE ST 2094-50 standard defines dynamic metadata for use in high
dynamic range (HDR) image sequences. This metadata provides information
for tone mapping of image content to displays characterized by their
HDR headroom.

## New Structures

* Extending VkHdrMetadataEXT

  * `VkHdrSmpteSt_2094_50_EXT`

```C
typedef struct VkHdrSmpteSt_2094_50_EXT {
  VkStructureType                          sType;
  const void*                              pNext;
  size_t                                   metadataSize;
  const void*                              metadataAddress;
  VkHdrSmpteSt_2094_50_PowerPreference_EXT powerPreference;
  VkHdrSmpteSt_2094_50_HdrHeadroomEXT      hdrHeadroom;
  float                                    hdrHeadroomLimit;
} VkHdrSmpteSt_2094_50_EXT;
```

* The `metadataSize` member indicates the size of the buffer pointed to by
  `metadataAddress`.
* The `metadataAddress` member points to a buffer containing the
  `smpte_st_2094_50_application_info()` bitstream, as defined in Annex C.2.1 of
  SMPTE ST 2094-50. This data is copied during the call to
  `vkSetHdrMetadataExt`, and the buffer may be modified or freed after the call.
  No validation is performed on the contents of this buffer.
* The `powerPreference` member of the `VkHdrSmpteSt_2094_50_EXT` structure
  allows an application to suggest a trade-off between color accuracy and
  power consumption.
  * When set to `VK_HDR_SMPTE_ST_2094_50_PREFER_COLOR_ACCURACY_EXT`,
    the implementation must use the tone mapping algorithm specified in
    Annex A of SMPTE ST 2094-50.
  * When set to `VK_HDR_SMPTE_ST_2094_50_PREFER_LOW_POWER_EXT`,
    the implementation may use simplified tone mapping algorithms to reduce
    power consumption, while still maintaining accuracy for greyscale content
    and providing reasonable behavior for color content (e.g, by using ICtCp
    or maxRGB tone mapping, as described in Step 5 of Annex 5 of
    [ITU-R BT.2408-8](https://www.itu.int/pub/R-REP-BT.2408-8-2024)).
* The `hdrHeadroom` member of the  `VkHdrSmpteSt_2094_50_EXT` structure
  allows an application to control the targeted HDR headroom for tone mapping.
  * When set to `VK_HDR_SMPTE_ST_2094_50_HDR_HEADROOM_NATIVE_EXT`,
    the implementation will tone map to the display's native HDR headroom.
  * When set to `VK_HDR_SMPTE_ST_2094_50_HDR_HEADROOM_LIMITED_EXT`,
    the implementation will tone map to the minimium of the display's native
    HDR headroom and the specified `hdrHeadroomLimit` member.

## New Enum Constants

* Extending VkStructureType:

  * `VK_STRUCTURE_TYPE_HDR_SMPTE_ST_2094_50_METADATA_EXT`

```C
typedef enum VkHdrSmpteSt_2094_50_PowerPreferenceEXT
  VK_HDR_SMPTE_ST_2094_50_PREFER_COLOR_ACCURACY_EXT = 0,
  VK_HDR_SMPTE_ST_2094_50_PREFER_LOW_POWER_EXT = 1,
} VkHdrSmpteSt_2094_50_PowerPreferenceEXT;

typedef enum VkHdrSmpteSt_2094_50_HdrHeadroomEXT
  VK_HDR_SMPTE_ST_2094_50_HDR_HEADROOM_NATIVE_EXT = 0,
  VK_HDR_SMPTE_ST_2094_50_HDR_HEADROOM_LIMITED_EXT = 1,
} VkHdrSmpteSt_2094_50_HdrHeadroomEXT;
```

## Version History

* Revision1, 2026-05-06 (Christopher Cameron)
  * Initial version
