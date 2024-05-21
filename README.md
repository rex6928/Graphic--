# 그래픽스 과제 
오늘 학습한 내용을 참고해서 Dev-CPP 사각형 큐브 대신 다른 도형 넣기,
MFC에서 사각형, 타원, 글씨에 다른 3가지 추가하기를 구현하고
github에 올리고, 링크와 소감을 제출하시오.

## 1. Dev-CPP 사각형 큐브 대신 오각형 큐브 넣기

```
/**************************
 * Includes
 *
 **************************/

#include <windows.h>
#include <gl/gl.h>
#include <math.h>

/**************************
 * Function Declarations
 *
 **************************/

LRESULT CALLBACK WndProc(HWND hWnd, UINT message, WPARAM wParam, LPARAM lParam);
void EnableOpenGL(HWND hWnd, HDC *hDC, HGLRC *hRC);
void DisableOpenGL(HWND hWnd, HDC hDC, HGLRC hRC);

/**************************
 * WinMain
 *
 **************************/

int WINAPI WinMain(HINSTANCE hInstance,
                   HINSTANCE hPrevInstance,
                   LPSTR lpCmdLine,
                   int iCmdShow)
{
    WNDCLASS wc;
    HWND hWnd;
    HDC hDC;
    HGLRC hRC;
    MSG msg;
    BOOL bQuit = FALSE;
    float theta = 0.0f;

    /* register window class */
    wc.style = CS_OWNDC;
    wc.lpfnWndProc = WndProc;
    wc.cbClsExtra = 0;
    wc.cbWndExtra = 0;
    wc.hInstance = hInstance;
    wc.hIcon = LoadIcon(NULL, IDI_APPLICATION);
    wc.hCursor = LoadCursor(NULL, IDC_ARROW);
    wc.hbrBackground = (HBRUSH)GetStockObject(BLACK_BRUSH);
    wc.lpszMenuName = NULL;
    wc.lpszClassName = "GLSample";
    RegisterClass(&wc);

    /* create main window */
    hWnd = CreateWindow(
        "GLSample", "OpenGL Sample",
        WS_CAPTION | WS_POPUPWINDOW | WS_VISIBLE,
        0, 0, 512, 512,
        NULL, NULL, hInstance, NULL);

    /* enable OpenGL for the window */
    EnableOpenGL(hWnd, &hDC, &hRC);

    /* program main loop */
    while (!bQuit)
    {
        /* check for messages */
        if (PeekMessage(&msg, NULL, 0, 0, PM_REMOVE))
        {
            /* handle or dispatch messages */
            if (msg.message == WM_QUIT)
            {
                bQuit = TRUE;
            }
            else
            {
                TranslateMessage(&msg);
                DispatchMessage(&msg);
            }
        }
        else
        {
            /* OpenGL animation code goes here */

            glClearColor(0.0f, 0.0f, 0.0f, 0.0f);
            glClear(GL_COLOR_BUFFER_BIT);

            glPushMatrix();
            glRotatef(theta, 0.0f, 0.0f, 1.0f);

            /* Draw filled pentagon */
            glBegin(GL_POLYGON);
            for (int i = 0; i < 5; ++i) {
                float angle = i * 2.0f * 3.14159f / 5;
                float x = 0.5f * cos(angle);
                float y = 0.5f * sin(angle);
                switch (i) {
                    case 0: glColor3f(1.0f, 0.0f, 0.0f); break;
                    case 1: glColor3f(0.0f, 1.0f, 0.0f); break;
                    case 2: glColor3f(0.0f, 0.0f, 1.0f); break;
                    case 3: glColor3f(1.0f, 1.0f, 0.0f); break;
                    case 4: glColor3f(0.0f, 1.0f, 1.0f); break;
                }
                glVertex2f(x, y);
            }
            glEnd();

            /* Draw white outline */
            glColor3f(1.0f, 1.0f, 1.0f);
            glBegin(GL_LINE_LOOP);
            for (int i = 0; i < 5; ++i) {
                float angle = i * 2.0f * 3.14159f / 5;
                float x = 0.5f * cos(angle);
                float y = 0.5f * sin(angle);
                glVertex2f(x, y);
            }
            glEnd();

            glPopMatrix();

            SwapBuffers(hDC);

            theta -= 1.0f;  // 반시계 방향 회전을 위해 감소
            Sleep(1);
        }
    }

    /* shutdown OpenGL */
    DisableOpenGL(hWnd, hDC, hRC);

    /* destroy the window explicitly */
    DestroyWindow(hWnd);

    return msg.wParam;
}

/********************
 * Window Procedure
 *
 ********************/

LRESULT CALLBACK WndProc(HWND hWnd, UINT message,
                          WPARAM wParam, LPARAM lParam)
{
    switch (message)
    {
    case WM_CREATE:
        return 0;
    case WM_CLOSE:
        PostQuitMessage(0);
        return 0;

    case WM_DESTROY:
        return 0;

    case WM_KEYDOWN:
        switch (wParam)
        {
        case VK_ESCAPE:
            PostQuitMessage(0);
            return 0;
        }
        return 0;

    default:
        return DefWindowProc(hWnd, message, wParam, lParam);
    }
}

/*******************
 * Enable OpenGL
 *
 *******************/

void EnableOpenGL(HWND hWnd, HDC *hDC, HGLRC *hRC)
{
    PIXELFORMATDESCRIPTOR pfd;
    int iFormat;

    /* get the device context (DC) */
    *hDC = GetDC(hWnd);

    /* set the pixel format for the DC */
    ZeroMemory(&pfd, sizeof(pfd));
    pfd.nSize = sizeof(pfd);
    pfd.nVersion = 1;
    pfd.dwFlags = PFD_DRAW_TO_WINDOW |
                  PFD_SUPPORT_OPENGL | PFD_DOUBLEBUFFER;
    pfd.iPixelType = PFD_TYPE_RGBA;
    pfd.cColorBits = 24;
    pfd.cDepthBits = 16;
    pfd.iLayerType = PFD_MAIN_PLANE;
    iFormat = ChoosePixelFormat(*hDC, &pfd);
    SetPixelFormat(*hDC, iFormat, &pfd);

    /* create and enable the render context (RC) */
    *hRC = wglCreateContext(*hDC);
    wglMakeCurrent(*hDC, *hRC);

    /* set the viewport */
    glViewport(0, 0, 512, 512);
}

/******************
 * Disable OpenGL
 *
 ******************/

void DisableOpenGL(HWND hWnd, HDC hDC, HGLRC hRC)
{
    wglMakeCurrent(NULL, NULL);
    wglDeleteContext(hRC);
    ReleaseDC(hWnd, hDC);
}

![캡처](https://github.com/rex6928/Graphic--/assets/162276764/abb00b5d-85e9-4487-bdfa-648cbfe9e232)


```

 ## 2. MFC에서 사각형, 타원, 글씨에 다른 3가지 추가하기

```

```















 

