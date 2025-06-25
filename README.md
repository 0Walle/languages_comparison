# Comparing how languages do things with array programming

Languages being compared:
 
 - C
 - Python
 - Julia
 - Haskell
 - APL

Some features:

| Language | Compre-<br>hensions | Array<br>Operators | Point Free  | High-Order<br>Operators | *N-Dim<br>Arrays* |
| -------- | -------------- | --------------- | -------------- | -------------------- | --- |
| C        | -              | -               | -              | -                    | - |
| Python   | **+**          | -               | -              | -                    | - |
| Julia    | **+**          | **+**           | -              | -                    | + |
| Haskell  | **+**          | -               | **+**          | -                    | - |
| APL      | **+**          | **+**           | **+**          | **+**                | + |

- C is here because is the classical example of imperative language.  
- Python is similar but with "modern" features and high adoption.  
- Julia is a modern language intended for scientific applications, which often involve N-Dim arrays so its a good candidate.  
- Haskell is the classical example of a functional language.  
- APL is here because its whole point is array programming, it's the language that started this project after all.  

## Image Blur (Convolution)

The image has been loaded with some external library, but they all consist of arrays of unsigned bytes with shape (Height, Width, Depth).

### APL (Dyalog APL)

```apl
∇ blur ← BlurImage img
    blur ← ({(+/∘⍉,[1 2]⍵)÷(KERNELSIZE×KERNELSIZE)}⌺KERNELSIZE KERNELSIZE) img
∇
```

### Python

Not tested

```python
def in_bounds(x, y, x_, y_):
    return 0 <= x_ < x and 0 <= y_ < y

def index(x, d, x_, y_, d_):
    return (y_*x+x_)*d+d_

def blur_image(input_bytes, output_bytes, x, y, depth):
    for y_ in range(y):
        for x_ in range(x):
            for d in range(depth):
                output_bytes[index(x, depth, x_, y_, d)] = int(sum(input_bytes[index(x, depth, x_+i, y_+j, d)] 
                    if in_bounds(x, y, x_+i, y_+j) else 0
                    for i in range(-KERNEL_SIZE//2,KERNEL_SIZE//2)
                    for j in range(-KERNEL_SIZE//2,KERNEL_SIZE//2)
                )/(KERNEL_SIZE*KERNEL_SIZE))
```

### C

```c
bool in_bounds(x, y, x_, y_)
{
    return 0 <= x_ && x_ < x && 0 <= y_ && y_ < y;
}

int index(x, d, x_, y_, d_)
{
    return (y_*x+x_)*d+d_;
}

void blur_image(unsigned char *input_bytes, unsigned char *output_bytes, int x, int y, int depth)
{
    for (int y_ = 0; y_ < y; y_++)
    for (int x_ = 0; x_ < x; x_++)
    for (int d = 0; d < depth; d++)
    {
        float sum = 0;
        for (int i = -KERNEL_SIZE/2; i <= KERNEL_SIZE/2; i++)
        for (int j = -KERNEL_SIZE/2; j <= KERNEL_SIZE/2; j++)
        {
            sum += in_bounds(x, y, x_+i, y_+j) ? input_bytes[index(x, depth, x_+i, y_+j, d)] : 0;
        }
        output_bytes[index(x, depth, x_, y_, d)] = (unsigned char)(sum/(KERNEL_SIZE*KERNEL_SIZE));
    }
}
```

### Haskell

Not tested

```hs
import Data.Word

inBounds x y x' y' = 0 <= x' && x' < x && 0 <= y' && y' < y

index bytes x d x' y' d' = bytes !! ((y'*x+x')*d+d')

blurImage :: [Word8] -> Int -> Int -> Int -> [Word8]
blurImage bytes x y d =
    [ avg [ index' (x'+i) (y'+j) d'
        | i <- [-kernelsize`div`2..kernelsize`div`2]
        , j <- [-kernelsize`div`2..kernelsize`div`2] ]
    | y' <- [0..y-1], x' <- [0..x-1], d' <- [0..d-1] ]
  where
    index' x' y' d'
      | inBounds x y x' y' = index bytes x d x' y' d'
      | otherwise = 0
    avg = ((kernelsize*kernelsize)`div`) . sum
```

### Julia

```julia
kernelsize = 3

function blurimage(input_bytes)
    r = div(kernelsize, 2)
    avg(xs) = div(sum(xs), (kernelsize*kernelsize))
    at(J) = checkbounds(Bool, input_bytes, J) ? input_bytes[J] : 0
    window(I) = avg(at.(CartesianIndices((-r:r,-r:r,0:0)).+I))
    window.(keys(input_bytes))
end
```
