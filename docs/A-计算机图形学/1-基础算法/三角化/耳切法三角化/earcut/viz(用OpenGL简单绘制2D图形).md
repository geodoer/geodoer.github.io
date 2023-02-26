## 简介
viz是earcut中一个简易的查看器，内置有很多测试用例。

1. 通过方向键左右，切换示例
2. 通过上下，切换`earcut`、`libtess2`的三角化结果

## 添加自己的示例
可以在创建窗口后，向测试用例容器中，添加自己的示例

```cpp
int main() 
{
	//...
    if (!window) {
        glfwTerminate();
        return 1;
    }

	//添自己的加测试用例
    {
        using collector = mapbox::fixtures::Collector<mapbox::fixtures::FixtureTester *>;
        collector::collection().clear(); //清空原来的示例

		//添加一个测试用例
        {
            static mapbox::fixtures::Fixture<short> my_test(
	            "mytest", //名称 
	            7, //期望得到几个三角形
	            1e-14,    //对earcut库计算结果，期望的偏差(Deviation)
	            0.000001, //对libtess库计算结果，期望的偏差
	            //仅绘制的话，前面的参数可以忽略，乱设置都行
                { //一个Polygon
                    { { 2, 2 }, { -2, 2 },{ -2, -2 },{ 2, -2 }, {2,2,} }, //外环
                    { { 1, 0 }, { 0, 1 }, { -1, 0 } } //内环
                    //还可以继续加内环
                });
            collector::add(&my_test);
        }
    }

	//...
}
```

## 源码阅读
要点

1. 只依赖了glfw，用于创建跨平台窗口
2. 用earcut、libtess2做了三角化，能够对比两者的效果
3. mvp矩阵都为单位矩阵，避免了引入数据库。二维空间下的坐标转换用加减乘除即可解决，无需用矩阵

总结：代码结构、逻辑清晰

```cpp
#include "comparison/earcut.hpp"
#include "comparison/libtess2.hpp"

#include "fixtures/geometries.hpp"

#if _MSC_VER >= 1900
#pragma comment(lib, "legacy_stdio_definitions.lib")
#endif

#include <GLFW/glfw3.h>

#include <cstdlib>
#include <cmath>
#include <vector>
#include <memory>

static GLFWwindow *window = nullptr;
static const int width = 1024;
static const int height = 1024;
static bool drawFill = true, drawMesh = true, drawOutline = true;
static bool dirtyViewport = true, dirtyShape = true, dirtyTessellator = true;
static float colorBackground[4] = {1.f, 1.f, 1.f, 1.f};
static float colorMesh[4] = {1.f, 0.f, 0.f, 0.2f}, colorFill[4] = {1.f, 1.f, 0.f, 0.2f};
static float colorOutline[4] = {0.2f, 0.2f, 0.2f, 0.9f}, colorInline[4] = {0.7f, 0.6f, .2f, 0.9f};

static bool mouseDrag = false;  //鼠标拖拽状态

static std::size_t shapeIndex = 0;  //当前使用的shape。下标
static std::size_t tessellator = 0; //当前使用哪个三角化工具。下标

// 2D的摄像机
struct Camera2D {
    double left = .0, right = .0, bottom = .0, top = .0;//Polygon的Box
    double translateX = .0, translateY = .0;            //平移值
    int viewWidth = 0, viewHeight = 0;                  //视口大小（像素单位）
    double zoom = -1.;                                  //缩放系数
    double mx = .0, my = .0;    //世界坐标数值 * mx = 屏幕坐标数值（屏幕坐标[-1,1]）
    double cx = .0, cy = .0;    //视口中心点的坐标（世界坐标到屏幕坐标下之后的偏移值），的负数
    inline float dpi() { return float(viewHeight) / height; }
    inline double scaling() { return std::pow(1.1, zoom); }
    void setView(int width, int height) {
        viewWidth = width;
        viewHeight = height;
    }
    void limits(double l, double r, double b, double t) {
        left = l;
        right = r;
        bottom = b;
        top = t;
    }
    bool scale(double z) {
        if (z == 0) return false;
        zoom += z;
        return true;
    }
    /*
     \param x,y: 像素单位。原点在左上角的屏幕坐标系
     */
    bool move(double x, double y) {
        const double s = scaling();
        const double dx = x / double(viewWidth) * (right - left) / s;
        const double dy = y / double(viewHeight) * (bottom - top) / s;
        if (dx == 0 && dy == 0) {
            return false;
        }
        translateX += dx;
        translateY += dy;
        return true;
    }
    /* apply current transform to opengl context */
    void apply() {
        glViewport(0, 0, viewWidth, viewHeight);

        //投影、模型视图矩阵都为单位矩阵
        //绘制时，改变坐标值即可
        //如此就不用依赖一个矩阵库
        glMatrixMode(GL_PROJECTION);
        glLoadIdentity();

        glMatrixMode(GL_MODELVIEW);
        glLoadIdentity();

        //计算 世界坐标->屏幕坐标 的变换参数
        const double s = scaling();
        const double w2 = .5 * (right - left);  //宽度的一半 
        const double h2 = .5 * (top - bottom);  //高度的一半
        mx = s / w2;                            //将世界坐标单位映射到屏幕坐标的单位上（mx=1/w2），多乘以了一个s，是因为要做一个缩放
        cx = (-left - w2 + translateX) * mx;    //(left+w2)*mx 中心点的x坐标值（屏幕坐标系）。这里多填了一个负号，则toScreen()方法中，就是+cx
        my = s / h2;
        cy = (-bottom - h2 + translateY) * my;
        /* todo: cx, cy这里取了负号，不方便理解
            改进方案：这里不乘-1，在计算时乘-1
                cx = (left + w2 - translateX) * mx;
                x = x * mx - cx;  //然后在toScreen中用减法
         */
        /* todo: 缩放逻辑放在这里一起计算，不方便理解。可以提到toScene中
                mx = 1 / w2;            //不在这里计算s
                x = (x * mx + cx) * s;  //在toScreen中做统一计算
         */ 
    }
    void setDefaults() {
        zoom = -1.;
        translateX = .0;
        translateY = .0;
    }
    //转到屏幕坐标空间，即映射到[-1, 1]范围
    /* transforms world coordinate to screen range [-1,1] with double precision */
    inline void toScreen(double *x, double *y) {
        *x = *x * mx + cx;  //*mx将世界坐标单位映射到屏幕坐标的单位; +cx即把坐标系原点移动到视口中心上
        *y = *y * my + cy;
    }
    /* transforms screen coordinates in range [-1,1] to world coordinates with double precision */
    inline void toWorld(double *x, double *y) {
        *x = (*x - cx) / mx;
        *y = (*y - cy) / my;
    }
    //世界坐标系 -> 屏幕坐标系，并绘制
    inline void vec2(double x, double y) {
        toScreen(&x, &y);
        glVertex2d(x, y); //todo: 摄像机中耦合了绘制逻辑
    }
};

static Camera2D cam;

// 绘制Polygon
class DrawablePolygon {
public:
    static std::unique_ptr<DrawablePolygon> makeDrawable(std::size_t index, mapbox::fixtures::FixtureTester* fixture);
    DrawablePolygon() = default;
    virtual ~DrawablePolygon() = default;
    //title
    virtual const char* name() = 0;
    //绘制三角化结果
    virtual void drawMesh() = 0;
    //绘制边框线
    virtual void drawOutline() = 0;
    //绘制Polygon的填充
    virtual void drawFill() = 0;
};

// 绘制三角化结果
class DrawableTesselator : public DrawablePolygon {
    //三角化结果
    mapbox::fixtures::FixtureTester::TesselatorResult shape;
    //Polygon
    mapbox::fixtures::DoublePolygon const& polygon;
public:
    explicit DrawableTesselator(mapbox::fixtures::FixtureTester::TesselatorResult tessellation,
                                mapbox::fixtures::DoublePolygon const& poly) : shape(tessellation), polygon(poly) { }
    //绘制Mesh（三角化结果的线框）
    void drawMesh() override {
        const auto &v = shape.vertices; //顶点
        const auto &x = shape.indices;  //索引
        glBegin(GL_LINES); //线段集
        glColor4fv(colorMesh);
        for (size_t i = 0; i < x.size(); i += 3) {
            //A->B
            cam.vec2(v[x[i]][0], v[x[i]][1]);
            cam.vec2(v[x[i + 1]][0], v[x[i + 1]][1]);
            //B->C
            cam.vec2(v[x[i + 1]][0], v[x[i + 1]][1]);
            cam.vec2(v[x[i + 2]][0], v[x[i + 2]][1]);
            //C->A
            cam.vec2(v[x[i + 2]][0], v[x[i + 2]][1]);
            cam.vec2(v[x[i]][0], v[x[i]][1]);
        }
        glEnd();
    }
    //绘制线框
    void drawOutline() override {
        glBegin(GL_LINES);
        for (std::size_t i = 0; i < polygon.size(); i++) {
            auto& ring = polygon[i];
            glColor4fv(i == 0 ? colorOutline : colorInline); //第一条是外环；其他是内环
            for (std::size_t j = 0; j < ring.size(); j++) {
                auto& p0 = ring[j];
                auto& p1 = ring[(j+1) % ring.size()];
                cam.vec2(std::get<0>(p0), std::get<1>(p0));
                cam.vec2(std::get<0>(p1), std::get<1>(p1));
            }
        }
        glEnd();
    }
    //绘制填充（三角化结果的三角面）
    void drawFill() override {
        const auto &v = shape.vertices;
        const auto &x = shape.indices;
        glBegin(GL_TRIANGLES);
        glColor4fv(colorFill);
        for (const auto pt : x) {
            cam.vec2(v[pt][0], v[pt][1]);
        }
        glEnd();
    }
};

// Earcut三角化
class DrawableEarcut : public DrawableTesselator {
public:
    explicit DrawableEarcut(mapbox::fixtures::FixtureTester* fixture)
            : DrawableTesselator(fixture->earcut(), fixture->polygon()) { }
    const char *name() override { return "earcut"; };
};

// libtess2三角化
class DrawableLibtess : public DrawableTesselator {
public:
    explicit DrawableLibtess(mapbox::fixtures::FixtureTester* fixture)
            : DrawableTesselator(fixture->libtess(), fixture->polygon()) { }
    const char *name() override { return "libtess2"; };
};

// 扫描线
class DrawableScanLineFill : public DrawablePolygon {
    struct Edge {
        float yMin;
        float yMax;
        float scale;
        float offset;
        Edge* next = nullptr;
        Edge(double x1, double y1, double x2, double y2)
                : yMin((float)std::min<double>(y1, y2)),
                  yMax((float)std::max<double>(y1, y2))
        {
            const double dx = x1 - x2;
            const double dy = y1 - y2;
            scale = (float)(dx / dy);
            offset = (float)((y1 * x2 - x1 * y2) / dy);
        }
        inline double intersection(double scanline) const {
            // horizontal scan-line intersection from the left
            // double precision for the calculation is required to avoid artifacts
            return scanline * scale + offset;
        }
    };
    mapbox::fixtures::FixtureTester* shape;
    std::vector<Edge*> activeList; /* contains current sorted intersections */
    std::vector<std::vector<Edge>> edgeTables; /* contains all edges sorted by yMin */
public:
    explicit DrawableScanLineFill(mapbox::fixtures::FixtureTester* fixture) : shape(fixture) {
        auto& polygon = shape->polygon();
        edgeTables.reserve(polygon.size());
        for (const auto &ring : polygon) {
            std::vector<Edge> edgeTable;
            edgeTable.reserve(ring.size());
            for (std::size_t i = 1; i <= ring.size(); i++) {
                const auto &p0 = ring[i - 1], p1 = i == ring.size() ? ring[0] : ring[i];
                const double x1 = std::get<0>(p0), y1 = std::get<1>(p0), x2 = std::get<0>(p1), y2 = std::get<1>(p1);
                if (y1 != y2) { edgeTable.emplace_back(x1, y1, x2, y2); }
            }
            std::sort(edgeTable.begin(), edgeTable.end(), [&](Edge const &a, Edge const &b) {
                return a.yMin < b.yMin;
            });
            edgeTables.push_back(std::move(edgeTable));
        }
    }
    void scanLineFill(std::vector<Edge>& edgeTable, double top, double bottom, double lineWidth) {
        assert(top < bottom);
        assert(lineWidth > 0);

        // create intrusive sorted edge list
        for (std::size_t i = 1; i < edgeTable.size(); i++) {
            edgeTable[i-1].next = &edgeTable[i];
        }
        Edge* edgeList = edgeTable.empty() ? nullptr : &edgeTable[0];

        const double halfWidth = lineWidth * 0.5;
        top -= lineWidth;
        bottom += lineWidth;

        double y0 = top;
        while(y0 < bottom && edgeList) {
            double y1 = y0 + lineWidth;
            if (y1 == y0) { y1 = std::nextafter(y1, bottom); }
            const double y = y0 + halfWidth;

            activeList.clear();
            Edge* prevEdge = nullptr;
            for (auto edge = edgeList; edge != nullptr; edge = edge->next) {
                const auto min = edge->yMin, max = edge->yMax;
                if (min <= y && y < max) {
                    activeList.push_back(edge);
                }
                if (min > y)  {
                    break;
                } else if (y >= max) {
                    // unlink edge
                    Edge** next = prevEdge ? &prevEdge->next : &edgeList;
                    *next = edge->next;
                } else {
                    prevEdge = edge;
                }
            }
            std::sort(activeList.begin(), activeList.end(), [&](Edge *a, Edge *b) {
                return a->intersection(y) < b->intersection(y);
            });

            double x1y0 = 0, x1y1 = 0;
            for (std::size_t i = 0; i < activeList.size(); i++) {
                Edge *edge = activeList[i];

                // use slope to make MSAA possible
                const double x2y0 = edge->intersection(std::max<double>(edge->yMin, y0));
                const double x2y1 = edge->intersection(std::min<double>(edge->yMax, y1));
                if ((i % 2) != 0) {
                    cam.vec2(x1y0, y0);
                    cam.vec2(x1y1, y1);
                    cam.vec2(x2y1, y1);
                    cam.vec2(x2y0, y0);
                }
                x1y0 = x2y0;
                x1y1 = x2y1;
            }

            y0 = y1;
        }
    }
    void drawFill() override {
        for (std::size_t i = 0; i < edgeTables.size(); i++) {
            auto& edgeTable = edgeTables[i];
            glBegin(GL_QUADS);
            glColor4fv(i == 0 ? colorFill : colorBackground);
            double x0 = 1;
            double y0 = 1;
            cam.toWorld(&x0, &y0);
            double x1 = -1;
            double y1 = -1;
            cam.toWorld(&x1, &y1);
            scanLineFill(edgeTable, y0, y1, (y1 - y0) / cam.viewHeight);
            glEnd();
        }
    }
    void drawOutline() override {
        auto& polygon = shape->polygon();
        glBegin(GL_LINES);
        for (std::size_t i = 0; i < polygon.size(); i++) {
            auto& ring = polygon[i];
            glColor4fv(i == 0 ? colorOutline : colorInline);
            for (std::size_t j = 0; j < ring.size(); j++) {
                auto& p0 = ring[j];
                auto& p1 = ring[(j+1) % ring.size()];
                cam.vec2(std::get<0>(p0), std::get<1>(p0));
                cam.vec2(std::get<0>(p1), std::get<1>(p1));
            }
        }
        glEnd();
    }
    void drawMesh() override { }
    const char *name() override { return "scanline-fill"; }
};

std::unique_ptr<DrawablePolygon> DrawablePolygon::makeDrawable(std::size_t index, mapbox::fixtures::FixtureTester* fixture) {
    if (index == 0) {
        return std::unique_ptr<DrawablePolygon>(new DrawableEarcut(fixture));
    } else if (index == 1) {
        return std::unique_ptr<DrawablePolygon>(new DrawableLibtess(fixture));
    } else {
        return std::unique_ptr<DrawablePolygon>(new DrawableScanLineFill(fixture));
    }
}

// 三角化工具
static std::array<std::unique_ptr<DrawablePolygon>, 3> tessellators;

// 获得测试数据
mapbox::fixtures::FixtureTester *getFixture(std::size_t i) {
    auto& fixtures = mapbox::fixtures::FixtureTester::collection();
    if (fixtures.empty()) {
        assert(false);
        exit(1);
    }
    return fixtures[i % fixtures.size()];
}

int main() {
    if (!glfwInit()) {
        return 1;
    }

    glfwWindowHint(GLFW_RESIZABLE, 0);
    glfwWindowHint(GLFW_SAMPLES, 4);
    window = glfwCreateWindow(width, height, "Tessellation", nullptr, nullptr);
    if (!window) {
        glfwTerminate();
        return 1;
    }

    // 键盘按下回调
    glfwSetKeyCallback(window,
                       [](GLFWwindow *win, int key, int /*scancode*/, int action, int /*mods*/) {
        if (action != GLFW_PRESS && action != GLFW_REPEAT) {
            return;
        }

        if (key == GLFW_KEY_ESCAPE || key == GLFW_KEY_Q) {
            //ESC, Q    退出
            glfwSetWindowShouldClose(win, 1);
        } else if (key == GLFW_KEY_F) {
            //F         绘制填充的开关
            drawFill = !drawFill;
            dirtyViewport = true;
        } else if (key == GLFW_KEY_M) {
            //M         绘制Mesh的开关
            drawMesh = !drawMesh;
            dirtyViewport = true;
        } else if (key == GLFW_KEY_O) {
            //O         绘制轮廓的边框
            drawOutline = !drawOutline;
            dirtyViewport = true;
        } else if (key == GLFW_KEY_RIGHT) {
            //方向键-右     切换shape
            if (shapeIndex + 1 < mapbox::fixtures::FixtureTester::collection().size()) {
                shapeIndex++;
                dirtyShape = true;
            }
        } else if (key == GLFW_KEY_LEFT) {
            //方向键-左     切换shape
            if (shapeIndex >= 1) {
                shapeIndex--;
                dirtyShape = true;
            }
        } else if (key == GLFW_KEY_T || key == GLFW_KEY_UP) {
            //方向键-上     切换三角化工具
            tessellator = (tessellator + 1) % tessellators.size();
            dirtyTessellator = true;
        } else if (key == GLFW_KEY_DOWN) {
            //方向键-下     切换三角化工具
            tessellator = (tessellator + tessellators.size() - 1) % tessellators.size();
            dirtyTessellator = true;
        } else if (key == GLFW_KEY_KP_ADD) {
            //右侧小键盘+号   缩放
            dirtyViewport |= cam.scale(1.);
        } else if (key == GLFW_KEY_KP_SUBTRACT) {
            //右侧小键盘-号   缩放
            dirtyViewport |= cam.scale(-1.);
        } else if (key == GLFW_KEY_R) {
            //R             重置，所有都绘制
            dirtyTessellator = dirtyViewport = dirtyShape = true;
        } else if (key == GLFW_KEY_W) {
            //w a s d       上下左右移动，控制摄像机
            dirtyViewport |= cam.move(.0, cam.viewHeight / 50.);
        } else if (key == GLFW_KEY_A) {
            dirtyViewport |= cam.move(cam.viewWidth / 50., .0);
        } else if (key == GLFW_KEY_S) {
            dirtyViewport |= cam.move(.0, -cam.viewHeight / 50.);
        } else if (key == GLFW_KEY_D) {
            dirtyViewport |= cam.move(-cam.viewWidth / 50., .0);
        }
    });

    // 鼠标中键滚动回调
    glfwSetScrollCallback(window, [](GLFWwindow* /* window */, double /* xoffset */, double yoffset) {
        dirtyViewport |= cam.scale(yoffset);
    });

    // 鼠标按键回调
    glfwSetMouseButtonCallback(window, [](GLFWwindow* /* window */, int button, int action, int /* mods */){
        if (button == GLFW_MOUSE_BUTTON_LEFT) {
            //鼠标左键
            mouseDrag = action != GLFW_RELEASE; //没有放开，则drag=true
        }
    });

    // 窗口大小改变回调
    glfwSetFramebufferSizeCallback(window, [](GLFWwindow * /*win*/, int w, int h) {
        cam.setView(w, h);
    });

    // 获得窗口大小，并赋值给照相机
    int fbWidth, fbHeight;
    glfwGetFramebufferSize(window, &fbWidth, &fbHeight);
    cam.setView(fbWidth, fbHeight);

    // 激活window为当前上下文
    glfwMakeContextCurrent(window);

    glfwSwapInterval(1);

    glClearColor(colorBackground[0], colorBackground[1], colorBackground[2], colorBackground[3]);

    glEnable(GL_BLEND);
    glBlendFunc(GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA);

    glEnable(GL_LINE_SMOOTH);
    glHint(GL_LINE_SMOOTH_HINT, GL_NICEST);

    while (!glfwWindowShouldClose(window)) {
        static double mouseX = 0, mouseY = 0;               //鼠标位置
        double mousePrevX = mouseX, mousePrevY = mouseY;    //上一个鼠标位置
        glfwGetCursorPos(window, &mouseX, &mouseY);         //获取当前鼠标位置（原点在左上角的屏幕坐标系）
        double mouseDeltaX = mouseX - mousePrevX, mouseDeltaY = mouseY - mousePrevY;    //鼠标位移量
        if (mouseDrag) {
            //鼠标左键按下 + 鼠标位置更改 => 移动摄像机
            dirtyViewport |= cam.move(mouseDeltaX, mouseDeltaY);
        }

        //shape弄脏 => 重新绘制
        if (dirtyShape) {
            //将三角化工具置空
            for (auto &tessellator : tessellators) {
                tessellator.reset(nullptr);
            }

            //生成数据，并获取polygon
            const auto& polygon = getFixture(shapeIndex)->polygon();

            //获得polygon的box
            auto minX = std::numeric_limits<double>::max();
            auto maxX = std::numeric_limits<double>::min();
            auto minY = std::numeric_limits<double>::max();
            auto maxY = std::numeric_limits<double>::min();
            if (!polygon.empty()) {
                for (const auto &pt : polygon[0]) {
                    minX = std::min<double>(minX, std::get<0>(pt));
                    minY = std::min<double>(minY, std::get<1>(pt));
                    maxX = std::max<double>(maxX, std::get<0>(pt));
                    maxY = std::max<double>(maxY, std::get<1>(pt));
                }
            }
            const auto dimX = minX < maxX ? maxX - minX : 0;
            const auto dimY = minY < maxY ? maxY - minY : 0;

            //中心点
            auto midX = minX + dimX / 2;
            auto midY = minY + dimY / 2;
            //box最大半径
            auto ext = std::max<double>(dimX, dimY) / 2;
            
            //设置摄像机
            cam.setDefaults();
            cam.limits(midX - ext, midX + ext, midY + ext, midY - ext);
        }

        if (dirtyViewport || dirtyShape || dirtyTessellator) {
            glClear(GL_COLOR_BUFFER_BIT);

            cam.apply();
            //线的宽度
            glLineWidth(cam.dpi() * std::sqrt(2.f));

            //三角化工具
            auto& drawable = tessellators[tessellator];

            if (!drawable) {
                //获得三角化工具
                drawable = DrawablePolygon::makeDrawable(tessellator, getFixture(shapeIndex));
            }

            if (dirtyTessellator || dirtyShape) {
                //不同三角化工具的title不同，因此要重新设置
                glfwSetWindowTitle(window, (std::string(drawable->name()) + ": "
                                            + getFixture(shapeIndex)->name).c_str());
            }

            if (!drawMesh && !drawFill && !drawOutline) {
                drawMesh = drawFill = drawOutline = true;
            }

            //绘制
            if (drawFill) {
                drawable->drawFill();
            }
            if (drawMesh) {
                drawable->drawMesh();
            }
            if (drawOutline) { //外边框最后绘制，以压盖三角化结果的边框
                drawable->drawOutline();
            }

            glFlush(); /* required for Mesa 3D driver */
            glfwSwapBuffers(window);
        }

        dirtyTessellator = dirtyShape = dirtyViewport = false;
        glfwWaitEvents();
    }

    glfwTerminate();
    return 0;
}
```