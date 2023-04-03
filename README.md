OpenH264 heap overflow

Details:
```
static inline int32_t FmoGenerateMbAllocMapType0 (PFmo pFmo, PPps pPps) {
   ...
   do {
    uint8_t uiGroup = 0;
    do {
[1]   const int32_t kiRunIdx = pPps->uiRunLength[uiGroup];
      int32_t j = 0;
      do {
        pFmo->pMbAllocMap[i + j] = uiGroup;
        ++ j;
      } while (j < kiRunIdx && i + j < iMbNum);
[2]   i += kiRunIdx;
      ++ uiGroup;
    } while (uiGroup < uiNumSliceGroups && i < iMbNum);
  } while (i < iMbNum);

 ...
}

```

We control the contents of pPps->uiRunLength array (see #1).

On line #2 we could make the value of 'i' variable negative if kiRunIdx will be less than 0.

Which will lead to pMbAllocMap overflow.

``
$ git clone https://github.com/cisco/openh264
$ cd openh264
$ USE_ASAN=Yes make
$ ./h264dec 1.264
```
