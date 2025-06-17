# 🎨 OpenGL 사각형 그리기

이 코드는 Windows API와 OpenGL을 직접 사용해서 컬러 사각형을 렌더링하는 예제입니다.
---

## 💻 코드 설명

### 1. 라이브러리 포함 및 윈도우 메시지 처리 함수

```cpp
#pragma comment(lib, "opengl32.lib")
#include <windows.h>
#include <gl/GL.h>

LRESULT CALLBACK WndProc(HWND h, UINT m, WPARAM w, LPARAM l) {
    if (m == WM_DESTROY) { PostQuitMessage(0); return 0; }
    return DefW
```
### 2. 윈도우 클래스 등록 및 창 생성

```cpp
int WINAPI WinMain(HINSTANCE hInst, HINSTANCE, LPSTR, int nCmd) {
    WNDCLASS wc = { CS_OWNDC, WndProc, 0, 0, hInst, 0, 0, 0, 0, L"GL" };
    RegisterClass(&wc);

    HWND hwnd = CreateWindow(L"GL", L"Rectangle",
        WS_OVERLAPPEDWINDOW, 100, 100, 800, 600,
        0, 0, hInst, 0);

    ShowWindow(hwnd, nCmd);

```

### 3. OpenGL 렌더링 컨텍스트 초기화
OpenGL에서 그리기 위해 윈도우 디바이스 컨텍스트와 렌더링 컨텍스트를 연결합니다.
```cpp
HDC dc = GetDC(hwnd);

    PIXELFORMATDESCRIPTOR pfd = { sizeof(pfd), 1,
        PFD_DRAW_TO_WINDOW | PFD_SUPPORT_OPENGL | PFD_DOUBLEBUFFER,
        PFD_TYPE_RGBA };

    SetPixelFormat(dc, ChoosePixelFormat(dc, &pfd), &pfd);

    HGLRC rc = wglCreateContext(dc);
    wglMakeCurrent(dc, rc);

```

### 4. 정점 데이터 정의 (위치와 색상)
정점 배열로 위치와 RGB 색상 데이터를 함께 저장합니다.
```cpp
    float v[] = {
        -0.5f, -0.5f, 1, 0, 0,  // 좌하 (빨강)
         0.5f, -0.5f, 0, 1, 0,  // 우하 (초록)
         0.5f,  0.5f, 0, 0, 1,  // 우상 (파랑)

        -0.5f, -0.5f, 1, 0, 0,  // 좌하 (빨강)
         0.5f,  0.5f, 0, 0, 1,  // 우상 (파랑)
        -0.5f,  0.5f, 1, 1, 0   // 좌상 (노랑)
    };

```

### 5. 메시지 루프와 렌더링 코드
OpenGL 함수로 정점과 색상 배열을 설정하고 삼각형 2개를 그립니다.
```cpp
 MSG msg;
    while (PeekMessage(&msg, 0, 0, 0, PM_REMOVE) || 1) {
        if (msg.message == WM_QUIT) break;

        TranslateMessage(&msg);
        DispatchMessage(&msg);

        glClearColor(0.2f, 0.3f, 0.3f, 1);
        glClear(GL_COLOR_BUFFER_BIT);

        glEnableClientState(GL_VERTEX_ARRAY);
        glEnableClientState(GL_COLOR_ARRAY);

        glVertexPointer(2, GL_FLOAT, 5 * sizeof(float), v);
        glColorPointer(3, GL_FLOAT, 5 * sizeof(float), v + 2);

        glDrawArrays(GL_TRIANGLES, 0, 6);

        SwapBuffers(dc);
    }

```

### 6. 종료 처리
프로그램 종료 시 렌더링 컨텍스트와 디바이스 컨텍스트를 정리합니다.
```cpp
 wglMakeCurrent(0, 0);
    wglDeleteContext(rc);
    ReleaseDC(hwnd, dc);
    return 0;
}
```



# 🎨 FreeGLUT를 이용하여 주전자 회전시키기

FreeGLUT를 이용해 빨간색 와이어프레임 주전자를 회전시키는 간단한 예제입니다.

---
## 💻 코드 설명

### 1. 헤더 포함과 전역 변수 선언

```cpp
#include <GL/freeglut.h>

float angle = 0.0f;
```

### 2. 화면 출력 함수 (display)
카메라를 설정하고 빨간 와이어프레임 주전자를 회전시켜 그립니다.
```cpp
void display() {
    glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);

    glMatrixMode(GL_MODELVIEW);
    glLoadIdentity();
    gluLookAt(0, 0, 3,  
              0, 0, 0,
              0, 1, 0);  

    glRotatef(angle, 0.0f, 1.0f, 0.0f);

    glColor3f(1.0f, 0.0f, 0.0f);

    glutWireTeapot(0.5);  

    glutSwapBuffers();
}
```
### 3. 타이머 함수 (timer)
매 프레임마다 회전 각도를 증가시키고 화면 갱신을 요청합니다.
```cpp
void timer(int value) {
    angle += 1.0f;
    if (angle > 360.0f) angle -= 360.0f;
    glutPostRedisplay();
    glutTimerFunc(16, timer, 0);
}
```
### 4. 창 크기 변경 처리 함수 (reshape)
창 크기가 변경될 때 올바른 원근 투영과 뷰포트를 설정합니다.
```cpp
void reshape(int w, int h) {
    if (h == 0) h = 1;
    float ratio = (float)w / (float)h;

    glMatrixMode(GL_PROJECTION);
    glLoadIdentity();
    gluPerspective(45.0, ratio, 1.0, 100.0);

    glViewport(0, 0, w, h);
}
```
### 5. main 함수 및 초기화
FreeGLUT 초기화 후 콜백 함수를 등록하고, 메인 루프를 시작합니다.
```cpp
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
