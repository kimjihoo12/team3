# 무지개 사각형
이 코드는 Windows API와 OpenGL을 직접 사용해서 컬러 사각형을 렌더링하는 예제다. FreeGLUT 같은 외부 라이브러리는 안 쓰고, windows.h와 GL.h를 사용한다. 좌표와 색상이 하나의 배열 v에 interleaved 형태로 들어있다.
(x, y) 좌표 두 개 다음에 (r, g, b) 세 개가 붙는다. 총 6개의 꼭짓점이 있으니까 삼각형 두 개가 만들어진다.
```cpp
#pragma comment(lib, "opengl32.lib")

#include <windows.h>
#include <gl/GL.h>

LRESULT CALLBACK WndProc(HWND h, UINT m, WPARAM w, LPARAM l) {
    if (m == WM_DESTROY) { PostQuitMessage(0); return 0; }
    return DefWindowProc(h, m, w, l);
}

int WINAPI WinMain(HINSTANCE hInst, HINSTANCE hPrevInstance, LPSTR lpCmdLine, int nCmd) {
    WNDCLASS wc = { CS_OWNDC, WndProc, 0, 0, hInst, 0, 0, 0, 0, L"GL" };
    RegisterClass(&wc);
    HWND hwnd = CreateWindow(L"GL", L"Rectangle", WS_OVERLAPPEDWINDOW, 100, 100, 800, 600, 0, 0, hInst, 0);
    ShowWindow(hwnd, nCmd);

    HDC dc = GetDC(hwnd);
    PIXELFORMATDESCRIPTOR pfd = { sizeof(pfd), 1, PFD_DRAW_TO_WINDOW | PFD_SUPPORT_OPENGL | PFD_DOUBLEBUFFER, PFD_TYPE_RGBA };
    SetPixelFormat(dc, ChoosePixelFormat(dc, &pfd), &pfd);
    HGLRC rc = wglCreateContext(dc); wglMakeCurrent(dc, rc);

    float v[] = {
        -0.5f, -0.5f, 1, 0, 0,
         0.5f, -0.5f, 0, 1, 0,
         0.5f,  0.5f, 0, 0, 1,

         -0.5f, -0.5f, 1, 0, 0,
          0.5f,  0.5f, 0, 0, 1,
         -0.5f,  0.5f, 1, 1, 0
    };

    MSG msg;
    while (PeekMessage(&msg, 0, 0, 0, PM_REMOVE) || 1) {
        if (msg.message == WM_QUIT) break;
        TranslateMessage(&msg); DispatchMessage(&msg);

        glClearColor(0.2f, 0.3f, 0.3f, 1); glClear(GL_COLOR_BUFFER_BIT);
        glEnableClientState(GL_VERTEX_ARRAY); glEnableClientState(GL_COLOR_ARRAY);
        glVertexPointer(2, GL_FLOAT, 5 * sizeof(float), v);
        glColorPointer(3, GL_FLOAT, 5 * sizeof(float), v + 2);
        glDrawArrays(GL_TRIANGLES, 0, 6);
        SwapBuffers(dc);
    }

    wglMakeCurrent(0, 0); wglDeleteContext(rc); ReleaseDC(hwnd, dc);
    return 0;
}
```
# 회전하는 주전자
이 코드는 FreeGLUT 라이브러리를 사용해서 화면 중앙에 빨간색 와이어프레임 주전자가 회전하는 3D 장면을 만든다. glRotatef()로 y축을 기준으로 주전자를 회전시키고, glColor3f(1,0,0)로 빨간색을 지정한다. glutWireTeapot(0.5)를 호출하면 와이어프레임 주전자가 그려진다.

```cpp
#include <GL/freeglut.h>

float angle = 0.0f;

void display() {
    glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);

    glMatrixMode(GL_MODELVIEW);
    glLoadIdentity();
    gluLookAt(0, 0, 3,   // eye
        0, 0, 0,   // center
        0, 1, 0);  // up

    glRotatef(angle, 0.0f, 1.0f, 0.0f);

    // 색상 설정: 빨간색
    glColor3f(1.0f, 0.0f, 0.0f);

    // 와이어프레임 주전자
    glutWireTeapot(0.5);

    glutSwapBuffers();
}

void timer(int value) {
    angle += 1.0f;
    if (angle > 360.0f) angle -= 360.0f;
    glutPostRedisplay();
    glutTimerFunc(16, timer, 0);
}

void reshape(int w, int h) {
    if (h == 0) h = 1;
    float ratio = (float)w / (float)h;

    glMatrixMode(GL_PROJECTION);
    glLoadIdentity();
    gluPerspective(45.0, ratio, 1.0, 100.0);

    glViewport(0, 0, w, h);
}

int main(int argc, char** argv) {
    glutInit(&argc, argv);
    glutInitDisplayMode(GLUT_DOUBLE | GLUT_RGB | GLUT_DEPTH);
    glutInitWindowSize(800, 600);
    glutCreateWindow("Red Wireframe Teapot");

    glEnable(GL_DEPTH_TEST);

    glutDisplayFunc(display);
    glutReshapeFunc(reshape);
    glutTimerFunc(0, timer, 0);

    glutMainLoop();
    return 0;
}
```
