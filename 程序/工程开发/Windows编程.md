```
#include <windows.h>
#include <GL/glew.h>
#include <GL/wglew.h>
#include <string.h>
#include <stdlib.h>
#include "winfw.h"
#define CLASSNAME L"EJOY"
#define WINDOWNAME L"EJOY"
#define WINDOWSTYLE (WS_OVERLAPPEDWINDOW & ~WS_THICKFRAME & ~WS_MAXIMIZEBOX)
static void
set_pixel_format_to_hdc(HDC hDC)
{
    int color_deep;
    PIXELFORMATDESCRIPTOR pfd;
    color_deep = GetDeviceCaps(hDC, BITSPIXEL);
    
    memset(&pfd, 0, sizeof(pfd));
    pfd.nSize = sizeof(pfd);
    pfd.nVersion = 1;
    pfd.dwFlags = PFD_DRAW_TO_WINDOW | PFD_SUPPORT_OPENGL | PFD_DOUBLEBUFFER;
    pfd.iPixelType  = PFD_TYPE_RGBA;
    pfd.cColorBits  = color_deep;
    pfd.cDepthBits  = 0;
    pfd.cStencilBits = 0;
    pfd.iLayerType  = PFD_MAIN_PLANE;
    int pixelFormat = ChoosePixelFormat(hDC, &pfd);
    SetPixelFormat(hDC, pixelFormat, &pfd);
}
static void
init_window(HWND hWnd) {
    HDC hDC = GetDC(hWnd);
    set_pixel_format_to_hdc(hDC);
    HGLRC glrc = wglCreateContext(hDC);
    if (glrc == 0) {
        exit(1);
    }
    wglMakeCurrent(hDC, glrc);
    if ( glewInit() != GLEW_OK ) {
        exit(1);
    }
    glViewport(0, 0, WIDTH, HEIGHT);
    ReleaseDC(hWnd, hDC);
}
static void
update_frame(HDC hDC) {
    ejoy2d_win_frame();
    SwapBuffers(hDC);
}
static void
get_xy(LPARAM lParam, int *x, int *y) {
    *x = (short)(lParam & 0xffff); 
    *y = (short)((lParam>>16) & 0xffff); 
}
static LRESULT CALLBACK
WndProc(HWND hWnd, UINT message, WPARAM wParam, LPARAM lParam)
{
    switch (message) {
    case WM_CREATE:
        init_window(hWnd);
        SetTimer(hWnd,0,10,NULL);
        break;
    case WM_PAINT: {
        if (GetUpdateRect(hWnd, NULL, FALSE)) {
            HDC hDC = GetDC(hWnd);
            update_frame(hDC);
            ValidateRect(hWnd, NULL);
            ReleaseDC(hWnd, hDC);
        }
        return 0;
    }
    case WM_TIMER : {
        ejoy2d_win_update();
        InvalidateRect(hWnd, NULL , FALSE);
        break;
    }
    case WM_DESTROY:
        PostQuitMessage(0);
        return 0;
    case WM_LBUTTONUP: {
        int x,y;
        get_xy(lParam, &x, &y); 
        ejoy2d_win_touch(x,y,TOUCH_END);
        break;
    }
    case WM_LBUTTONDOWN: {
        int x,y;
        get_xy(lParam, &x, &y); 
        ejoy2d_win_touch(x,y,TOUCH_BEGIN);
        break;
    }
    case WM_MOUSEMOVE: {
        int x,y;
        get_xy(lParam, &x, &y); 
        ejoy2d_win_touch(x,y,TOUCH_MOVE);
        break;
    }
    }
    return DefWindowProcW(hWnd, message, wParam, lParam);
}
static void
register_class()
{
    WNDCLASSW wndclass;
    wndclass.style = CS_HREDRAW | CS_VREDRAW | CS_OWNDC;
    wndclass.lpfnWndProc = WndProc;
    wndclass.cbClsExtra = 0;
    wndclass.cbWndExtra = 0;
    wndclass.hInstance = GetModuleHandleW(0);
    wndclass.hIcon = 0;
    wndclass.hCursor = 0;
    wndclass.hbrBackground = 0;
    wndclass.lpszMenuName = 0; 
    wndclass.lpszClassName = CLASSNAME;
    RegisterClassW(&wndclass);
}
static HWND
create_window(int w, int h) {
    RECT rect;
    rect.left=0;
    rect.right=w;
    rect.top=0;
    rect.bottom=h;
    AdjustWindowRect(&rect,WINDOWSTYLE,0);
    HWND wnd=CreateWindowExW(0,CLASSNAME,WINDOWNAME,
        WINDOWSTYLE, CW_USEDEFAULT,0,
        rect.right-rect.left,rect.bottom-rect.top,
        0,0,
        GetModuleHandleW(0),
        0);
    return wnd;
}
int
main(int argc, char *argv[]) {
    register_class();
    HWND wnd = create_window(WIDTH,HEIGHT);
    ejoy2d_win_init(argc, argv);
    ShowWindow(wnd, SW_SHOWDEFAULT);
    UpdateWindow(wnd);
    MSG msg;
    while (GetMessage(&msg, NULL, 0, 0)) {
        TranslateMessage(&msg);
        DispatchMessage(&msg);
    }
    return 0;
}
```

* **register_class()**  注册窗口类
	* RegisterClassW，注册 unicode 窗口类还有个RegisterClassA，是ASCII码窗口 
	* lpfnWndProc = WndProc; 设置事件回调
	* wndclass.lpszClassName = CLASSNAME; 设置窗口类名，之后就用这个类名来访问窗口类
* **create_window** 创建窗口
	* AdjustWindowRect(&rect,WINDOWSTYLE,0); 传入的rect是窗口的客户区，也就是除去标题栏，关闭按钮等等区域，调整得到新的rect，表示要显示的客户区所需窗口大小。
	* CreateWindowExW 创建窗口
* **ShowWindow(wnd, SW_SHOWDEFAULT);** 显示窗口
* **UpdateWindow(wnd);**  立即发送一个WM_PAINT 消息，不需要GetMessage 也能响应
* **init_window** 初始化OpenGL 上下文
	* set_pixel_format_to_hdc  设置设备属性
		* PIXELFORMATDESCRIPTOR pfd;  PFD_DRAW_TO_WINDOW | PFD_SUPPORT_OPENGL | PFD_DOUBLEBUFFER; 采用 OpenGL 绘图，开启双缓冲
		* pfd.iPixelType  = PFD_TYPE_RGBA;   像素格式
		* pfd.cColorBits  = color_deep;  像素大小
		* pfd.cDepthBits  = 0;   是否开启深度缓存
		* pfd.cStencilBits = 0; 是否开启模版缓存
		* ChoosePixelFormat
		* SetPixelFormat
	* wglCreateContext 创建 OpenGL 上下文
	* wglMakeCurrent 设置为当前上下文，之后就可以直接用 OpenGL 的 API 了。
	* glewInit 初始化glew，在Windows下使用OpenGL必然要用到glew，因为Windows 自带的OpenGL只支持到1.1。glew可以帮我们检测哪些 API 可以用。
* **ValidateRect(hWnd, NULL);**  使某些客户区有效，传递NULL参数意味着使所有客户区有效。并删除WM_PAINT消息。
* **InvalidateRect(hWnd, NULL , FALSE);**  使某些客户区，传递NULL参数意味着所有客户区无效，会发送WM_PAINT到窗体，如果客户区一直是无效的，那就会不停的发 WM_PAINT 消息给窗体。
* **GetUpdateRect**  获得无效区域