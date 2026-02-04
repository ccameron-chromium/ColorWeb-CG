# SMPTE ST 2094-50 integration

## Introduction

This proposal suggests a mechanism to integrate SMPTE ST 2094-50 tone mapping metadata into

* [`CanvasRenderingContext2D`](https://html.spec.whatwg.org/multipage/canvas.html#2dcontext),
  [`OffscreenCanvasRenderingContext2D`](https://html.spec.whatwg.org/multipage/canvas.html#the-offscreen-2d-rendering-context),
  [`WebGLRenderingContextBase`](https://registry.khronos.org/webgl/specs/latest/1.0/#5.14), and
  [`CanvasContext`](https://www.w3.org/TR/webgpu/#canvas-context)
* [`VideoFrameMetadata`](https://www.w3.org/TR/webcodecs/#dictdef-videoframemetadata) in WebCodecs.

### Motivation

SMPTE ST 2094-50 provides headroom-adaptive tone mapping.

The public committee draft for SMPTE ST 2094-50 may be found [here](https://github.com/SMPTE/st2094-50).
The second public committee draft (under development) will include the `smpte_st_2094_50_application_info` bitstream for the metadata.

This bitstream can be embeded in a video stream (via multiplexed timed metadata tracks or ITU-T T.35 supplemental encoding information messages) or in an image (via ICC profiles).

### Related work

The various rendering context APIs (will, after adoption of [this proposal](https://github.com/ccameron-chromium/ColorWeb-CG/blob/canvas2d_hdr_headroom/canvas2d_tone_map.md)) use the `CanvasToneMapping` and `CanvasToneMappingMode` interface to specify their tone mapping.

```idl
  enum CanvasToneMappingMode {
      "standard",
      "extended",
  };

  dictionary CanvasToneMapping {
    CanvasToneMappingMode mode = "standard";
  };
```

The WebCodecs API interface to retrieve video frame metadata, and to specify video frame metadata for encoding.

```idl
  partial interface VideoFrame {
      VideoFrameMetadata metadata();
  };

  dictionary VideoFrameMetadata {
    // Possible members are recorded in the VideoFrame Metadata Registry.
  };
```

The [VideoFrame Metadata Registry](https://w3c.github.io/webcodecs/video_frame_metadata_registry.html) contains a list of metadata entries.

## Proposal for Canvas APIs

To the `CanvasToneMapping` structure, add an entry for `smpte_st_2094_50_application_info` metadata bitstream.

```idl
  dictionary CanvasToneMapping {
    CanvasToneMappingMode mode;
    Blob smpteSt2094_50ApplicationInfo;
  }
```

To the `CanvasToneMappingMode` enum, add a mode indicating that SMPTE ST 2094-50 metadata is to be used for tone mapping.

```idl
  enum CanvasToneMappingMode {
      "standard",
      "extended",
      "smpte-st-2094-50",
  };
```

When the canvas has its tone mapping's `mode` set to `"smpte-st-2094-50"`, then the contents of its `smpte_st_2094_50` dictionary will be for tone mapping.

Images that result from calls to `toBlob` and `toDataURL` shall have the SMPTE ST 2094-50 metadata embedded in their ICC profile.

Videos that result from calls to [`captureStream`](https://webrtc.github.io/samples/src/content/capture/canvas-video/) shall have the SMPTE ST 2094-50 metadata attached.

## Proposal for WebCodecs API

Add a new specification to the [VideoFrame Metadata Registry](https://w3c.github.io/webcodecs/video_frame_metadata_registry.html).

This specification will add a new entry for the `smpte_st_2094_50_application_info` metadata bitstream.

```idl
  partial dictionary {
    SmpteSt_2094_50_ColorVolumeTransform smpte_st_2094_50_color_volume_transform;
  }

  dictionary SmpteSt_2094_50_ColorVolumeTransform {
    float hdrReferenceWhite;
    optional SmpteSt_2094_50_HeadroomAdaptiveToneMap headroomAdaptiveToneMap;
  }
  dictionary SmpteSt_2094_50_HeadroomAdaptiveToneMap {
    float baselineHdrHeadroom;
    sequence<SmpteSt_2094_50_AlternateImage> alternateImages;
    SmpteSt_2094_50_ColorPrimaries gainApplicationSpaceColorPrimaries;
  }
  dictionary SmpteSt_2094_50_AlternateImage {
    float hdrHeadroom;
    SmpteSt_2094_50_ColorGainFunction colorGainFunction;
  }
  dictionary SmpteSt_2094_50_ColorGainFunction {
    SmpteSt_2094_50_ComponentMix componentMix;
    SmpteSt_2094_50_GainCurve gainCurve;
  }
  dictionary SmpteSt_2094_50_ComponentMix {
    float red;
    float green;
    float blue;
    float min;
    float max;
    float channel;
  }
  dictionary SmpteSt_2094_50_GainCurve {
    sequence<SmpteSt_2094_50_ControlPoint> controlPoints;
  }
  dictionary SmpteSt_2094_50_ControlPoint {
    float x;
    float y;
    float m;
  }
  dictionary SmpteSt_2094_50_ColorPrimaries {
    SmpteSt_2094_50_ChromaticityCoordinates red;
    SmpteSt_2094_50_ChromaticityCoordinates green;
    SmpteSt_2094_50_ChromaticityCoordinates blue;
    SmpteSt_2094_50_ChromaticityCoordinates white;
  }
  dictionary SmpteSt_2094_50_ChromaticityCoordinates {
    float x;
    float y;
  }
```

## Interactions with ImageBitmap APIs

An `ImageBitmap` that is created from a source that has SMPTE ST 2094-50 metadata should preserve that metadata.
This metadata, like the pixels and color space of the `ImageBitmap`, is not directly accessible, but affects operations performed with the `ImageBitmap`.

When the `ImageBitmap` is rendered using an `ImageBitmapRenderingContext`, the metadata should be used for tone mapping.
In particular, the resulting `<canvas>` element will render the same as an `<img>` element with the same source.

When the `ImageBitmap` is drawn to a 2D bitmap at a specific headroom (using `drawImage`, `texImage`, or `copyExternalImageToTexture`), the tone mapping performed should be that of the source's metadata.
In particular, the result should be the same as if the operation had been performed directly with the source.

When the `ImageBitmap` is exported as an image or video, the resulting encoding should preserve the metadata.

