
## Steps

- [x] Line detection to run with multiprocessing (2024.09.10, saves 3s)
- [ ] Build OpenCV with GPU support
- [ ] Investigate imports time spend
- [ ] Logging: save images asynchronously
- [x] Run image denoising, edge detection and line detection on cropped images only (2024.09.19, saves 1.5s)
- [x] Ternary search for solution, start at 486 with step size 243, scale by 3 (2024.09.19, saves 0.2s)
- [ ] Multiprocessing on distance measurement
- [ ] Coordinate with Jexin: Annotated data to be sent asynchronously
- [ ] Error reporting to be done asynchronously

## Present breakdown:

| Step | Step time | Cumulative time | GPU acceleration expected | Other improvements expected | Other comments |
| --- | --- | --- | --- | --- | --- |
| Imports | 0.243 | 0.243 |     |     | Surprisingly high, needs investigation, but counts as setup time and not cycle time |
| Function definitions | 0.001 | 0.244 |     |     |     |
| Loading input data | 0.437 | 0.681 |     |     | Needs investigation, likely culprit is reading a 20MB image from disk |
| Processing profile data | 0.001 | 0.681 |     |     |     |
| Image calibration | 0.309 | 0.991 | Yes |     | Pure OpenCV function |
| Initialize logging | 0.044 | 1.035 |     | Yes | Culprit: saving a 4MB image to disk |
| Image de-noising | 0 (distributed into line detection) | 1.035 | Yes |     | Pure OpenCV function |
| Edge detection | 0 (distributed into line detection) | 1.035 | Yes |     | Pure OpenCV function, but can be broken down to smaller images |
| Line detection | 0.306 | 1.341 | Yes |     | Multiprocessing already implemented |
| Measuring distances between line intersections | 0.021 | 1.363 |     | Yes | Multiprocessing possible here |
| Create annotated output image | 0.988 | 2.351 | Yes | Yes | Can be sent asynchronously |
| Reporting “no errors” | 0.006 | 2.358 |     |     | Low because of network proximity. If this changes, this needs to be done asynchronously too. |
| Put together response object | 0.002 | 2.359 |     |     |     |

Total cycle time: 1.678s (excluding setup time in imports and function definitions) (excluding time taken to read image from disk because in practice the image will be read from RAM)

Total time of steps where only GPU acceleration is expected: 0.309s  
Total time of steps (cycle time only) where other enhancements are expected: 0.065s  
Total time of steps where GPU acceleration and other enhancements are expected: 0.988s