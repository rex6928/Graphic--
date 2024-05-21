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


```

![캡처](https://github.com/rex6928/Graphic--/assets/162276764/abb00b5d-85e9-4487-bdfa-648cbfe9e232)

 ## 2. MFC에서 사각형, 타원, 글씨에 다른 3가지 추가하기

```
// MFCApplication2.cpp: 애플리케이션에 대한 클래스 동작을 정의합니다.
//

#include "pch.h"
#include "framework.h"
#include "afxwinappex.h"
#include "afxdialogex.h"
#include "MFCApplication2.h"
#include "MainFrm.h"

#include "MFCApplication2Doc.h"
#include "MFCApplication2View.h"

#ifdef _DEBUG
#define new DEBUG_NEW
#endif


// CMFCApplication2App

BEGIN_MESSAGE_MAP(CMFCApplication2App, CWinApp)
    ON_COMMAND(ID_APP_ABOUT, &CMFCApplication2App::OnAppAbout)
    // 표준 파일을 기초로 하는 문서 명령입니다.
    ON_COMMAND(ID_FILE_NEW, &CWinApp::OnFileNew)
    ON_COMMAND(ID_FILE_OPEN, &CWinApp::OnFileOpen)
    // 표준 인쇄 설정 명령입니다.
    ON_COMMAND(ID_FILE_PRINT_SETUP, &CWinApp::OnFilePrintSetup)
END_MESSAGE_MAP()


// CMFCApplication2App 생성

CMFCApplication2App::CMFCApplication2App() noexcept
{
    // 다시 시작 관리자 지원
    m_dwRestartManagerSupportFlags = AFX_RESTART_MANAGER_SUPPORT_ALL_ASPECTS;
#ifdef _MANAGED
    // 애플리케이션을 공용 언어 런타임 지원을 사용하여 빌드한 경우(/clr):
    //     1) 이 추가 설정은 다시 시작 관리자 지원이 제대로 작동하는 데 필요합니다.
    //     2) 프로젝트에서 빌드하려면 System.Windows.Forms에 대한 참조를 추가해야 합니다.
    System::Windows::Forms::Application::SetUnhandledExceptionMode(System::Windows::Forms::UnhandledExceptionMode::ThrowException);
#endif

    // TODO: 아래 애플리케이션 ID 문자열을 고유 ID 문자열로 바꾸십시오(권장).
    // 문자열에 대한 서식: CompanyName.ProductName.SubProduct.VersionInformation
    SetAppID(_T("MFCApplication2.AppID.NoVersion"));

    // TODO: 여기에 생성 코드를 추가합니다.
    // InitInstance에 모든 중요한 초기화 작업을 배치합니다.
}

// 유일한 CMFCApplication2App 개체입니다.

CMFCApplication2App theApp;


// CMFCApplication2App 초기화

BOOL CMFCApplication2App::InitInstance()
{
    // 애플리케이션 매니페스트가 ComCtl32.dll 버전 6 이상을 사용하여 비주얼 스타일을
    // 사용하도록 지정하는 경우, Windows XP 상에서 반드시 InitCommonControlsEx()가 필요합니다. 
    // InitCommonControlsEx()를 사용하지 않으면 창을 만들 수 없습니다.
    INITCOMMONCONTROLSEX InitCtrls;
    InitCtrls.dwSize = sizeof(InitCtrls);
    // 응용 프로그램에서 사용할 모든 공용 컨트롤 클래스를 포함하도록
    // 이 항목을 설정하십시오.
    InitCtrls.dwICC = ICC_WIN95_CLASSES;
    InitCommonControlsEx(&InitCtrls);

    CWinApp::InitInstance();


    // OLE 라이브러리를 초기화합니다.
    if (!AfxOleInit())
    {
        AfxMessageBox(IDP_OLE_INIT_FAILED);
        return FALSE;
    }

    AfxEnableControlContainer();

    EnableTaskbarInteraction(FALSE);

    // RichEdit 컨트롤을 사용하려면 AfxInitRichEdit2()가 있어야 합니다.
    // AfxInitRichEdit2();

    // 표준 초기화
    // 이들 기능을 사용하지 않고 최종 실행 파일의 크기를 줄이려면
    // 아래에서 필요 없는 특정 초기화
    // 루틴을 제거해야 합니다.
    // 해당 설정이 저장된 레지스트리 키를 변경하십시오.
    // TODO: 이 문자열을 회사 또는 조직의 이름과 같은
    // 적절한 내용으로 수정해야 합니다.
    SetRegistryKey(_T("로컬 애플리케이션 마법사에서 생성된 애플리케이션"));
    LoadStdProfileSettings(4);  // MRU를 포함하여 표준 INI 파일 옵션을 로드합니다.


    // 애플리케이션의 문서 템플릿을 등록합니다.  문서 템플릿은
    //  문서, 프레임 창 및 뷰 사이의 연결 역할을 합니다.
    CSingleDocTemplate* pDocTemplate;
    pDocTemplate = new CSingleDocTemplate(
        IDR_MAINFRAME,
        RUNTIME_CLASS(CMFCApplication2Doc),
        RUNTIME_CLASS(CMainFrame),       // 주 SDI 프레임 창입니다.
        RUNTIME_CLASS(CMFCApplication2View));
    if (!pDocTemplate)
        return FALSE;
    AddDocTemplate(pDocTemplate);


    // 표준 셸 명령, DDE, 파일 열기에 대한 명령줄을 구문 분석합니다.
    CCommandLineInfo cmdInfo;
    ParseCommandLine(cmdInfo);



    // 명령줄에 지정된 명령을 디스패치합니다.
    // 응용 프로그램이 /RegServer, /Register, /Unregserver 또는 /Unregister로 시작된 경우 FALSE를 반환합니다.
    if (!ProcessShellCommand(cmdInfo))
        return FALSE;

    // 창 하나만 초기화되었으므로 이를 표시하고 업데이트합니다.
    m_pMainWnd->ShowWindow(SW_SHOW);
    m_pMainWnd->UpdateWindow();
    return TRUE;
}

int CMFCApplication2App::ExitInstance()
{
    //TODO: 추가한 추가 리소스를 처리합니다.
    AfxOleTerm(FALSE);

    return CWinApp::ExitInstance();
}

// CMFCApplication2App 메시지 처리기


// 응용 프로그램 정보에 사용되는 CAboutDlg 대화 상자입니다.

class CAboutDlg : public CDialogEx
{
public:
    CAboutDlg() noexcept;

    // 대화 상자 데이터입니다.
#ifdef AFX_DESIGN_TIME
    enum { IDD = IDD_ABOUTBOX };
#endif

protected:
    virtual void DoDataExchange(CDataExchange* pDX);    // DDX/DDV 지원입니다.
    virtual BOOL OnInitDialog(); // 다이얼로그 초기화

    // 구현입니다.
protected:
    DECLARE_MESSAGE_MAP()

private:
    CRect m_rect;
    CRect m_ellipse;
    CPoint m_textPosition;
    CRect m_additionalRect1;
    CRect m_additionalRect2;
    CRect m_additionalRect3;
    CPoint m_lastPoint;
    bool m_isFirstMove;

public:
    afx_msg void OnMouseMove(UINT nFlags, CPoint point);
    afx_msg void OnPaint();
};

CAboutDlg::CAboutDlg() noexcept : CDialogEx(IDD_ABOUTBOX), m_isFirstMove(true)
{
    // Initialize the shapes' initial positions and sizes
    m_rect = CRect(50, 50, 150, 150);
    m_ellipse = CRect(200, 50, 300, 150);
    m_textPosition = CPoint(350, 100);
    m_additionalRect1 = CRect(50, 200, 150, 300);
    m_additionalRect2 = CRect(200, 200, 300, 300);
    m_additionalRect3 = CRect(350, 200, 450, 300);
}

void CAboutDlg::DoDataExchange(CDataExchange* pDX)
{
    CDialogEx::DoDataExchange(pDX);
}

BOOL CAboutDlg::OnInitDialog()
{
    CDialogEx::OnInitDialog();

    // 센터 초기화
    CRect clientRect;
    GetClientRect(&clientRect);
    m_lastPoint = CPoint(clientRect.Width() / 2, clientRect.Height() / 2);

    return TRUE;
}

BEGIN_MESSAGE_MAP(CAboutDlg, CDialogEx)
    ON_WM_MOUSEMOVE()
    ON_WM_PAINT()
END_MESSAGE_MAP()

// 대화 상자를 실행하기 위한 응용 프로그램 명령입니다.
void CMFCApplication2App::OnAppAbout()
{
    CAboutDlg aboutDlg;
    aboutDlg.DoModal();
}

// CMFCApplication2App 메시지 처리기

void CAboutDlg::OnMouseMove(UINT nFlags, CPoint point)
{
    if (m_isFirstMove)
    {
        m_lastPoint = point;
        m_isFirstMove = false;
    }

    // Calculate the offset
    int offsetX = point.x - m_lastPoint.x;
    int offsetY = point.y - m_lastPoint.y;

    // Move the shapes
    m_rect.OffsetRect(offsetX, offsetY);
    m_ellipse.OffsetRect(offsetX, offsetY);
    m_textPosition.Offset(offsetX, offsetY);
    m_additionalRect1.OffsetRect(offsetX, offsetY);
    m_additionalRect2.OffsetRect(offsetX, offsetY);
    m_additionalRect3.OffsetRect(offsetX, offsetY);

    // Save the current mouse position for the next move
    m_lastPoint = point;

    // Invalidate the window to trigger a redraw
    Invalidate();

    CDialogEx::OnMouseMove(nFlags, point);
}

void CAboutDlg::OnPaint()
{
    CPaintDC dc(this); // device context for painting

    // Draw the rectangle
    dc.Rectangle(m_rect);

    // Draw the ellipse
    dc.Ellipse(m_ellipse);

    // Draw the text
    dc.TextOut(m_textPosition.x, m_textPosition.y, _T("Hello, MFC!"));

    // Draw additional shapes
    dc.Rectangle(m_additionalRect1);
    dc.Rectangle(m_additionalRect2);
    dc.Rectangle(m_additionalRect3);
}

```















 

