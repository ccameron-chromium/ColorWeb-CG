# VK_GOOGLE_hdr_SMPTE_ST_2094_50_metadata (3)

## Name

VK_GOOGLE_hdr_SMPTE_ST_2094_50_metadata - device extension

## VK_GOOGLE_hdr_SMPTE_ST_2094_50_metadata

### Name string

VK_GOOGLE_hdr_SMPTE_ST_2094_50_metadata

### Extension Type

Device extension

### Registered Extension Number

XXX

### Revision

XXX

### Ratification Status

Not ratified

### Extension and Version Dependencies

[VK_GOOGLE_hdr_metadata](https://docs.vulkan.org/refpages/latest/refpages/source/VK_GOOGLE_hdr_metadata.html)

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
`VkHdrSmpteSt2094_50_GOOGLE` structure.

The SMPTE ST 2094-50 standard defines dynamic metadata for use in high
dynamic range (HDR) image sequences. This metadata provides information
for tone mapping of image content to displays characterized by their
HDR headroom.

## New Structures

* Extending VkHdrMetadataGOOGLE

  * `VkHdrSmpteSt2094_50_GOOGLE`

```C
typedef struct VkHdrSmpteSt2094_50_GOOGLE {
  VkStructureType                           sType;
  const void*                               pNext;
  size_t                                    metadataSize;
  const void*                               metadataAddress;
  VkHdrSmpteSt2094_50_PowerPreferenceGOOGLE powerPreference;
  VkHdrSmpteSt2094_50_HdrHeadroomGOOGLE     hdrHeadroom;
  float                                     hdrHeadroomLimit;
} VkHdrSmpteSt2094_50_GOOGLE;
```

* The `metadataSize` member indicates the size of the buffer pointed to by
  `metadataAddress`.
* The `metadataAddress` member points to a buffer containing the
  `smpte_st_2094_50_application_info()` bitstream, as defined in Annex C.2.1 of
  SMPTE ST 2094-50. This data is copied during the call to
  `vkSetHdrMetadataExt`, and the buffer may be modified or freed after the call.
  No validation is performed on the contents of this buffer.
* The `powerPreference` member of the `VkHdrSmpteSt2094_50_GOOGLE` structure
  allows an application to suggest a trade-off between color accuracy and
  power consumption.
  * When set to `VK_HDR_SMPTE_ST_2094_50_PREFER_COLOR_ACCURACY_GOOGLE`,
    the implementation must use the tone mapping algorithm specified in
    Annex A of SMPTE ST 2094-50.
  * When set to `VK_HDR_SMPTE_ST_2094_50_PREFER_LOW_POWER_GOOGLE`,
    the implementation may use simplified tone mapping algorithms to reduce
    power consumption, while still maintaining accuracy for greyscale content
    and providing reasonable behavior for color content (e.g, by using ICtCp
    or maxRGB tone mapping, as described in Step 5 of Annex 5 of
    [ITU-R BT.2408-8](https://www.itu.int/pub/R-REP-BT.2408-8-2024)).
* The `hdrHeadroom` member of the  `VkHdrSmpteSt2094_50_GOOGLE` structure
  allows an application to control the targeted HDR headroom for tone mapping.
  * When set to `VK_HDR_SMPTE_ST_2094_50_HDR_HEADROOM_NATIVE_GOOGLE`,
    the implementation will tone map to the display's native HDR headroom.
  * When set to `VK_HDR_SMPTE_ST_2094_50_HDR_HEADROOM_LIMITED_GOOGLE`,
    the implementation will tone map to the minimium of the display's native
    HDR headroom and the specified `hdrHeadroomLimit` member.

## New Enum Constants

* Extending VkStructureType:

  * `VK_STRUCTURE_TYPE_HDR_SMPTE_ST_2094_50_METADATA_GOOGLE`

```C
typedef enum VkHdrSmpteSt2094_50_PowerPreferenceGOOGLE
  VK_HDR_SMPTE_ST_2094_50_PREFER_COLOR_ACCURACY_GOOGLE = 0,
  VK_HDR_SMPTE_ST_2094_50_PREFER_LOW_POWER_GOOGLE = 1,
} VkHdrSmpteSt2094_50_PowerPreferenceGOOGLE;

typedef enum VkHdrSmpteSt2094_50_HdrHeadroomGOOGLE
  VK_HDR_SMPTE_ST_2094_50_HDR_HEADROOM_NATIVE_GOOGLE = 0,
  VK_HDR_SMPTE_ST_2094_50_HDR_HEADROOM_LIMITED_GOOGLE = 1,
} VkHdrSmpteSt2094_50_HdrHeadroomGOOGLE;
```

## Version History

* Revision1, 2026-05-06 (Christopher Cameron)
  * Initial version
