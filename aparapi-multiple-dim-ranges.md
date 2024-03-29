# 多维执行域

> 如何在多维循环中使用 `Range`

Aparapi now allows developers to execute over one, two or three dimensional ranges. OpenCL natively allows the user to execute over 1, 2 or 3 dimension grids via the clEnqueueNDRangeKernel() method.

Initially we chose not to expose 2D or 3D ranges (Aparapi’s Kernel.execute(range) allowed only !d ranges, but following a specific request we added the notion of a Range via the new com.aparapi.Range class.

A range is created using various static factory methods. For example to create a simple range {0..1024} we would use.

Range range = Range.create(1024); In this case the range will span 1..1024 and a ‘default’ group size will be decided behind the scenes (256 probably in this case).

If the user wishes to select a specific group size (say 32) for a one dimensional Range (0..1024) then they can use.

Range range = Range.create(1024, 32); The group size must always be a ‘factor’ of the global range. So globalRange % groupSize == 0

For a 2D range we use the Range.create2D(…) factory methods.

Range range = Range.create2D(32, 32); The above represents a 2D grid of execution 32 rows by 32 columns. In this case a default group size will be determined by the runtime.

If we wish to specify the groupsize (say 4x4) then we can use.

```java
Range range = Range.create2D(32, 32, 4, 4);
This example uses a 2D range to apply a blurring convolution effect to a pixel buffer.

final static int WIDTH=128;
final static int HEIGHT=64;
final int in[] = new int[WIDTH*HEIGHT];
final int out[] = new int[WIDTH*HEIGHT];
Kernel kernel = new Kernel(){
   public void run(){
      int x = getGlobalId(0);
      int y = getGlobalId(1);
      if (x>0 && x<(getGlobalSize(0)-1) && y>0 && y<(getGlobalSize(0)-1)){
         int sum = 0;
         for (int dx =-1; dx<2; dx++){
           for (int dy =-1; dy<2; dy++){
             sum+=in[(y+dy)*getGlobalSize(0)+(x+dx)];
           }
         }
         out[y*getGlobalSize(0)+x] = sum/9;
      }
   }

};
Range range = Range.create2D(WIDTH, HEIGHT);
kernel.execute(range);
```

Handling this from JTP mode
Mapping to OpenCL for this is all fairly straightforward.

In Java JTP mode we have to emulate the execution over the 1D, 2D and 3D ranges using threads. Note that the number of threads we launch is essentially the size of the group. So be careful creating large groups.

If we ask for a 3D range using :-

```java
Range range = Range.create3D(1024, 1024, 1024, 8, 8, 8);
```

We are asking for a group size of 8x8x8 == 512. So we are asking for 512 threads!