# tesselator

C libaries: [GLU Libtess](https://gitlab.freedesktop.org/mesa/glu/tree/master/src/libtess) / [libtess2](https://github.com/memononen/libtess2). Check OpenGL SW/HW tessellator compare [here](https://stackoverflow.com/questions/11311698/shader-tessellation-vs-algorithmic-tessellation).


this is refactored version of the original libtess which comes with the GLU reference implementation.
this is C++ version(c++98), Using STL as memory pool.
Independent version without any third-party libraries, header files only.

重构版本的 gluTesselation 库，C++98 版本，使用了 STL 容器做内存缓存。
不依赖任何三方库，独立版本，只需要引用头文件就能使用。

琢磨了几个月，这次心血来潮，重构了代码，用了十来天时间（记不清了）。
原版本是个C库，到处是跳转和 #define，要把这些分散的函数封装成类，难度颇大，改动一处错误百出。
libtess2 作者对原代码重构过一次，但还是 C 版本，而且整体结构变动不大。
这次重构，看着 libtess2 的代码，又看着原版代码，为了查找错误，前后对照了几遍！吐血！
这次重构成 C++ 版本，是第一步，之后会进行后续优化。

也很纳闷，这么多年了，没有人重做一个OpenGL的三角形分解库，一直是 SGI 的这个 libtess。
GPC 简单易用，但是开源协议不友好；poly2tri 比 libtess快，但效果不如 libtess 好，有时候很不稳定。
希望我这个代码做个好的开头，有人能构建出更好更快的库，同时开源协议要友好。

For OpenGL, For Open source !!!

# main class:
<pre><code>
class Tesselator
{
  int init();
  void dispose();
  int add_contour( int size, const void* pointer, int stride, int count );
  int tesselate( TessWindingRule windingRule, TessElementType elementType, int polySize = 3);
};
</code></pre>
# Exsample:
<pre><code>

#define LIBTESS_USE_VEC2

#include <gl/glew.h>
#include <libtess/tess.hpp>

struct vec2f
{
    float x, y;
};

//tesselate polygons
libtess::Tesselator tess;
tess.init();

std::vector<Vec2> points;
//points.push_back(...)//add some points
tess.add_contour( 2, &points[0], sizeof(Vec2), points.size() );
//tess.AddContour( ... );
tess.Tesselate( libtess::TESS_WINDING_ODD, libtess::TESS_TRIANGLES );

//OpenGL drawing:
void draw_elements(int shape, const Vec2* vs, const int* indices, int size)
{
    #ifdef LIBTESS_HIGH_PRECISION
    glVertexPointer(2, GL_DOUBLE, sizeof(Vec2), vs);
    #else
    glVertexPointer(2, GL_FLOAT, sizeof(Vec2), vs);
    #endif
    glEnableClientState(GL_VERTEX_ARRAY);
    glDrawElements(shape, size, GL_UNSIGNED_INT, indices);
    glDisableClientState(GL_VERTEX_ARRAY);
}

draw_elements(CGL_TRIANGLES, &tess.vertices[0], &tess.elements[0], tess.elements.size());
</code></pre>

TESS_TRIANGLES执行结果:
![TESS_TRIANGLES执行结果](https://github.com/sdragonx/libtess/blob/master/TRIANGLES.png)
TESS_BOUNDARY_CONTOURS执行结果:
![TESS_BOUNDARY_CONTOURS执行结果](https://github.com/sdragonx/libtess/blob/master/BOUNDARY_CONTOURS.png)

