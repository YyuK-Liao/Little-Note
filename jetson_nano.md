## WHY
最近因為工作的關係，要將[YyuK-Liao/BRA](https://github.com/YyuK-Liao/BRA)搬到jetson nano上來測試，但NVIDIA對jetson nano維護的軟體版本都很久遠，所以這裡大多會記錄編譯的相關事項。

#### Python

#### Pytorch and torchvision
https://qengineering.eu/install-pytorch-on-jetson-nano.html

#### OpenCV
https://qengineering.eu/install-opencv-4.5-on-jetson-nano.html

#### TensorRT
https://github.com/NVIDIA/TensorRT/tree/main/python
https://github.com/mlcommons/inference_results_v2.0/issues/2#issuecomment-1133845520
https://stackoverflow.com/questions/58393546/compile-tensorrt-sample-not-found-nvonnxparsertypedefs-h

#### FFmpeg (NVIDIA build)
https://stackoverflow.com/questions/63479215/does-ffmpeg-support-gpu-acceleration-on-jetson-platform

#### Extra: LLVM14

#### Extra: pkg-config
https://jyhshin.pixnet.net/blog/post/26588033


## Q
```
OSError: /home/yuu/Downloads/BRA/.dev/lib/python3.8/site-packages/torch/lib/libgomp-d22c30c5.so.1: cannot allocate memory in static TLS block
```
https://forums.developer.nvidia.com/t/what-is-cannot-allocate-memory-in-static-tls-block/169225


```
    def Allocate(self)->Tuple[deque[np.ndarray], int, deque[int], bool]
                        │     │     │  │              └ <class 'collections.deque'>
                        │     │     │  └ <class 'numpy.ndarray'>
                        │     │     └ <module 'numpy' from '/home/yuu/Downloads/BRA/.dev/lib/python3.8/site-packages/numpy/__init__.py'>
                        │     └ <class 'collections.deque'>
                        └ typing.Tuple

TypeError: 'type' object is not subscriptable
```
用typing.Deque取代hint中的deque