# MYFIRSTCPROJECT
```c
#include<stdio.h>
#include<windows.h>
#include<conio.h>
#include<stdlib.h>
#include<time.h>
#include<time.h>
#pragma warning (disable:4996)

#define LEFT 75
#define RIGHT 77
#define UP 72
#define DOWN 80
#define LEFT2 97
#define RIGHT2 100
#define UP2 119
#define DOWN2 115
#define PAUSE 112           // 키보드의 아스키값
#define ESC 27              
#define SUBMIT 4
#define one 49
#define two 50

#define MAP_X 3
#define MAP_Y 2
#define MAP_WIDTH 40
#define MAP_HEIGHT 30

int x[2][100], y[2][100]; // 뱀 2마리의 좌표를 백개씩 저장
int dir1, dir2; // 두 플레이어의 이동 방향을 저장
int key; // 키를 입력받고 저장한다
int key2;
int food_x, food_y; // 음식의 좌표
int length[2]; // 뱀들의 길이를 저장
int speed; // 게임 스피드
int score;

void CursorView()
{
    CONSOLE_CURSOR_INFO cursorInfo = { 0, };
    cursorInfo.dwSize = 1; //커서 굵기 (1 ~ 100)
    cursorInfo.bVisible = //커서를 숨긴다.
        SetConsoleCursorInfo(GetStdHandle(STD_OUTPUT_HANDLE), &cursorInfo);
}
void gotoxy(int x, int y, char* s) { //x값을 2x로 변경, 좌표값에 바로 문자열을 입력할 수 있도록 printf함수 삽입  
    COORD pos = { 2 * x,y };
    SetConsoleCursorPosition(GetStdHandle(STD_OUTPUT_HANDLE), pos);
    printf("%s", s);    // 특정 위치에 원하는 문자열을 출력한다.
}
void gotoxy2(int x, int y) {
    printf("\033[%d;%df", y, x);
}

void title(void);   // 게임을 실행시키고 나타나는 화면
void countdown(void);   //게임 시작전 카운트 다운
void countdown_1(void); // 1인 선택시
void rule(void);
void reset(void);  // 게임을 리셋한다
void reset_1(void); //
void drawMap(void); // 게임 맵을 그린다
void drawMapTest(void);
void move1_1(int dir1); // 1인용 전용 움직임 제어 함수 
void move1(int dir1);   // 2인플레이의 첫번째 플레이어 (방향키로 이동)의 움직임을 제어
void move2(int dir2);   // 2인플레이의 두번째 플레이어 (WASD로 이동)의 움직임을 제어
void gameOver(void);    // 게임오버 조건을 확인 후 게임오버
void gameOver2(void);
void food(void);    // 음식을 생성
void cursor(int n); // 커서 숨기기
void chooseMod(void);   // 모드 선택
void keyScan1(void); // 키 입력
void keyScan2(void); // 2인용 키 입력
void gameLoop();
void ifgameover(void); // 게임 오버 되었을 시 모드 선택

enum ColorType {
    BLACK,
    darkBLUE,
    Green,
    darkSkyBlue,
    DarkRed,
    DarkPurple,
    DarkYellow,
    GRAY,
    DarkGray,
    BLUE,
    GREEN,
    SkyBlue,
    RED,
    PURPLE,
    YELLOW,
    WHITE,
    Aqua,
} COLOR;


int main(){
    system("mode con cols=100 lines=35");
    srand(time(NULL)); // 난수 발생기 초기화
    title();
    rule();
    chooseMod();
    system("pause");
    return 0;
}

void keyScan1(void) {
    reset_1();
    countdown_1();
    while (1) {
        if (kbhit()) {
            do {
                key = getch();
            } while (key == 224); //키 입력받음, 224 = 확장 키
        }
        Sleep(speed);

        switch (key) { //입력받은 키를 파악하고 실행  
        case LEFT:
        case RIGHT:
        case UP:
        case DOWN:
            if ((dir1 == LEFT && key != RIGHT) || (dir1 == RIGHT && key != LEFT) || (dir1 == UP && key != DOWN) ||
                (dir1 == DOWN && key != UP))//180회전이동을 방지하기 위해 필요. 
                dir1 = key;
            key = 0; // 키값을 저장하는 함수를 reset 
            break;
        case ESC: //ESC키를 누르면 프로그램 종료 
            exit(0);
        }
        move1_1(dir1);
    }
}

void keyScan2(void) {
    reset();
    countdown();
    while (1) {
        if (GetAsyncKeyState(VK_UP) & 0x8000 && dir1 != DOWN) {
            dir1 = UP;
        }
        if (GetAsyncKeyState(VK_DOWN) & 0x8000 && dir1 != UP) {
            dir1 = DOWN;
        }
        if (GetAsyncKeyState(VK_LEFT) & 0x8000 && dir1 != RIGHT) {
            dir1 = LEFT;
        }
        if (GetAsyncKeyState(VK_RIGHT) & 0x8000 && dir1 != LEFT) {
            dir1 = RIGHT;
        }
        if (GetAsyncKeyState(0x57) & 0x8000 && dir2 != DOWN) {
            dir2 = UP;
        }
        if (GetAsyncKeyState(0x53) & 0x8000 && dir2 != UP) {
            dir2 = DOWN;
        }
        if (GetAsyncKeyState(0x41) & 0x8000 && dir2 != RIGHT) {
            dir2 = LEFT;
        }
        if (GetAsyncKeyState(0x44) & 0x8000 && dir2 != LEFT) {
            dir2 = RIGHT;
        }

        move1(dir1);
        move2(dir2);

        Sleep(speed);
    }



}


void move1_1(int dir1) {
    int i;
    int prevX = x[0][0]; // 머리의 이전 X좌표를 저장한다.
    int prevY = y[0][0]; // 머리의 이전 Y좌표를 저장한다.
    if (x[0][0] == food_x && y[0][0] == food_y) { // 뱀의 음식과 충돌한 경우
        food(); // 새로운 음식 추가
        score += 10;
        length[0]++; // 뱀의 길이를 증가시킨다
        x[0][length[0] - 1] = x[0][length[0] - 2]; // 새로 생성된 몸통에 값을 할당
        y[0][length[0] - 1] = y[0][length[0] - 2];
    }

    // 뱀이 벽에 충돌한 경우를 확인
    if (x[0][0] == 0 || x[0][0] == MAP_WIDTH - 1 || y[0][0] == 0 || y[0][0] == MAP_HEIGHT - 1) {
        // 반대쪽 벽에서 나타남
        if (x[0][0] == 0)
            x[0][0] = MAP_WIDTH - 2;
        else if (x[0][0] == MAP_WIDTH - 1)
            x[0][0] = 1;
        if (y[0][0] == 0)
            y[0][0] = MAP_HEIGHT - 2;
        else if (y[0][0] == MAP_HEIGHT - 1)
            y[0][0] = 1;  // 벽을 통과하는 코드
        // 이전 머리위치를 지움
        gotoxy(MAP_X + prevX, MAP_Y + prevY, "  ");
    }

    for (i = 1; i < length[0]; i++) { // 자신과 충돌했는지 확인
        if (x[0][0] == x[0][i] && y[0][0] == y[0][i]) {
            gameOver();
            return;
        }
    }

    SetConsoleTextAttribute(GetStdHandle(STD_OUTPUT_HANDLE), DarkYellow);
    gotoxy(MAP_X + x[0][length[0] - 1], MAP_Y + y[0][length[0] - 1], "  "); // 마지막 몸통 부분을 지움
    for (i = length[0] - 1; i > 0; i--) { // 몸통 좌표를 한 칸씩 앞으로 이동
        x[0][i] = x[0][i - 1];
        y[0][i] = y[0][i - 1];
    }
    gotoxy(MAP_X + x[0][0], MAP_Y + y[0][0], "●"); // 이전 머리 위치에 몸통 부분을 그림
    if (dir1 == LEFT) --x[0][0]; // 새로운 머리 위치 (x[0], y[0])를 방향에 따라 변경
    if (dir1 == RIGHT) ++x[0][0];
    if (dir1 == UP) --y[0][0];
    if (dir1 == DOWN) ++y[0][0];
    gotoxy(MAP_X + x[0][0], MAP_Y + y[0][0], "⊙"); // 새로운 위치에 머리를 그림

    // 뱀을 움직인 후 맵(벽 포함)을 그립니다/
    drawMapTest();

    cursor(0); // 커서를 숨김
}

void move1(int dir1) {
    int i;
    int prevX = x[0][0]; // 이전 머리의 X 좌표 저장
    int prevY = y[0][0]; // 이전 머리의 Y 좌표 저장

    if (x[0][0] == food_x && y[0][0] == food_y) { // 뱀이 음식과 충돌한 경우
        food(); // 새로운 음식 추가
        length[0]++; // 뱀의 길이를 증가시킴
        x[0][length[0] - 1] = x[0][length[0] - 2]; // 새로 생성된 몸통에 값을 할당
        y[0][length[0] - 1] = y[0][length[0] - 2];
    }

    if (x[0][0] == 0 || x[0][0] == MAP_WIDTH - 1 || y[0][0] == 0 || y[0][0] == MAP_HEIGHT - 1) {
        gameOver();
        return;
    }

    for (i = 1; i < length[1]; i++) {
        if (x[0][0] == x[1][i] && y[0][0] == y[1][i]) {
            gameOver();
            return;
        }
    }


    //for (i = 1; i < length[0]; i++) { // 자신과 충돌했는지 확인
    //    if (x[0][0] == x[0][i] && y[0][0] == y[0][i]) {
    //        gameOver();
    //        return;
    //    }
    //}

    //for (i = 1; i < length[0]; i++) { // 자신과 충돌했는지 확인
    //    if (x[0][0] == x[1][i] && y[0][0] == y[1][i]) {
    //        gameOver();
    //        return;
    //    }
    //}

    SetConsoleTextAttribute(GetStdHandle(STD_OUTPUT_HANDLE), DarkYellow);
    gotoxy(MAP_X + x[0][length[0] - 1], MAP_Y + y[0][length[0] - 1], "  "); // 마지막 몸통 부분을 지움
    for (i = length[0] - 1; i > 0; i--) { // 몸통 좌표를 한 칸씩 앞으로 이동
        x[0][i] = x[0][i - 1];
        y[0][i] = y[0][i - 1];
    }

    // 이전 머리 위치에 몸통 부분을 그림
    gotoxy(MAP_X + prevX, MAP_Y + prevY, "●");

    // 새로운 머리 위치 (x[0], y[0])를 방향에 따라 변경
    if (dir1 == LEFT) --x[0][0];
    if (dir1 == RIGHT) ++x[0][0];
    if (dir1 == UP) --y[0][0];
    if (dir1 == DOWN) ++y[0][0];

    // 새로운 위치에 머리를 그림
    gotoxy(MAP_X + x[0][0], MAP_Y + y[0][0], "⊙");

    // 뱀을 움직인 후 맵(벽 포함)을 그림
    drawMap();

    cursor(0); // 커서를 숨김
}
void move2(int dir2) {
    int i;
    int prevX = x[1][0]; // 이전 머리의 X 좌표 저장
    int prevY = y[1][0]; // 이전 머리의 Y 좌표 저장

    if (x[1][0] == food_x && y[1][0] == food_y) { // 뱀이 음식과 충돌한 경우
        food(); // 새로운 음식 추가
        length[1]++; // 뱀의 길이를 증가시킴
        x[1][length[1] - 1] = x[1][length[1] - 2]; // 새로 생성된 몸통에 값을 할당
        y[1][length[1] - 1] = y[1][length[1] - 2];
    }

    if (x[1][0] == 0 || x[1][0] == MAP_WIDTH - 1 || y[1][0] == 0 || y[1][0] == MAP_HEIGHT - 1) {
        gameOver2();
        return;
    }

    for (i = 1; i < length[0]; i++) {
        if (x[1][0] == x[0][i] && y[1][0] == y[0][i]) {
            gameOver2();
            return;
        }
    }

    SetConsoleTextAttribute(GetStdHandle(STD_OUTPUT_HANDLE), RED);
    gotoxy(MAP_X + x[1][length[1] - 1], MAP_Y + y[1][length[1] - 1], "  "); // 마지막 몸통 부분을 지움
    for (i = length[1] - 1; i > 0; i--) { // 몸통 좌표를 한 칸씩 앞으로 이동
        x[1][i] = x[1][i - 1];
        y[1][i] = y[1][i - 1];
    }

    // 이전 머리 위치에 몸통 부분을 그림
    gotoxy(MAP_X + prevX, MAP_Y + prevY, "■");

    // 새로운 머리 위치 (x[1], y[1])를 방향에 따라 변경
    if (dir2 == LEFT) --x[1][0];
    if (dir2 == RIGHT) ++x[1][0];
    if (dir2 == UP) --y[1][0];
    if (dir2 == DOWN) ++y[1][0];

    // 새로운 위치에 머리를 그림
    gotoxy(MAP_X + x[1][0], MAP_Y + y[1][0], "⊙");

    // 뱀을 움직인 후 맵(벽 포함)을 그림
    drawMap();

    cursor(0); // 커서를 숨김
}








void gameLoop() {
    while (1) {
        if (kbhit()) { // 키 입력이 있는 경우
            int key = getch();
            if (key == 224) { // 확장 키 코드인 경우
                key = getch();
                if (key == 72 && dir2 != DOWN) // 위쪽 화살표 키이고 현재 방향이 아래쪽이 아닌 경우
                    dir2 = UP;
                else if (key == 80 && dir2 != UP) // 아래쪽 화살표 키이고 현재 방향이 위쪽이 아닌 경우
                    dir2 = DOWN;
                else if (key == 75 && dir2 != RIGHT) // 왼쪽 화살표 키이고 현재 방향이 오른쪽이 아닌 경우
                    dir2 = LEFT;
                else if (key == 77 && dir2 != LEFT) // 오른쪽 화살표 키이고 현재 방향이 왼쪽이 아닌 경우
                    dir2 = RIGHT;
            }
        }

        move1(dir1); // 첫 번째 뱀 이동 처리
        move2(GetAsyncKeyState(VK_UP) ? UP : GetAsyncKeyState(VK_DOWN) ? DOWN : GetAsyncKeyState(VK_LEFT) ? LEFT : GetAsyncKeyState(VK_RIGHT) ? RIGHT : dir2); // 두 번째 뱀 이동 처리

        // 게임 로직 추가

        drawMap(); // 맵 그리기
    }
}


void title(void) {
    int i, j;
    while (kbhit()) getch(); //버퍼에 있는 키값을 버림 
    reset();

    drawMap();    //맵 테두리를 그림 
    for (i = MAP_Y + 1; i < MAP_Y + MAP_HEIGHT - 1; i++) { // 맵 테두리 안쪽을 빈칸으로 채움 
        for (j = MAP_X + 1; j < MAP_X + MAP_WIDTH - 1; j++) gotoxy(j, i, "  ");
    }
    SetConsoleTextAttribute(GetStdHandle(STD_OUTPUT_HANDLE), RED);
    gotoxy(MAP_X + (MAP_WIDTH / 2) - 7, MAP_Y + 10, "+--------------------------+");
    gotoxy(MAP_X + (MAP_WIDTH / 2) - 7, MAP_Y + 11, "|                          |");
    gotoxy(MAP_X + (MAP_WIDTH / 2) - 7, MAP_Y + 12, "|  EXITING SNAKE GAME!!!!! |");
    gotoxy(MAP_X + (MAP_WIDTH / 2) - 7, MAP_Y + 13, "|                          |");
    gotoxy(MAP_X + (MAP_WIDTH / 2) - 7, MAP_Y + 14, "+--------------------------+ ");

    gotoxy(MAP_X + (MAP_WIDTH / 2) - 7, MAP_Y + 20, " < PRESS ANY KEY TO START > ");
    gotoxy(MAP_X + (MAP_WIDTH / 2) - 7, MAP_Y + 21, "       <정상혁,양동찬>");



    while (1) {
        if (kbhit()) { //키입력받음 
            key = getch();
            if (key == ESC) exit(0); // ESC키면 종료 
            else break; //아니면 그냥 계속 진행 
        }
        CursorView(0);
    }
    reset(); // 게임을 초기화  
}

void rule(void) {
    int i, j;

    while (kbhit()) getch(); //버퍼에 있는 키값을 버림 

    drawMap();    //맵 테두리를 그림 
    for (i = MAP_Y + 1; i < MAP_Y + MAP_HEIGHT - 1; i++) { // 맵 테두리 안쪽을 빈칸으로 채움 
        for (j = MAP_X + 1; j < MAP_X + MAP_WIDTH - 1; j++) gotoxy(j, i, "  ");
    }
    SetConsoleTextAttribute(GetStdHandle(STD_OUTPUT_HANDLE), RED);
    gotoxy(MAP_X + (MAP_WIDTH / 2) - 14, MAP_Y + 9,  "안녕하세요, exciting snake game!! 에 오신것을 환영합니다!");
    gotoxy(MAP_X + (MAP_WIDTH / 2) - 14, MAP_Y + 10, "                                                      ");
    gotoxy(MAP_X + (MAP_WIDTH / 2) - 14, MAP_Y + 11, "          저희 게임의 기반은 지렁이 게임이나 ");
    gotoxy(MAP_X + (MAP_WIDTH / 2) - 14, MAP_Y + 12, "          1인용 2인용이 나뉘어져 있습니다!                    ");
    gotoxy(MAP_X + (MAP_WIDTH / 2) - 14, MAP_Y + 13, "                                                      ");
    gotoxy(MAP_X + (MAP_WIDTH / 2) - 19, MAP_Y + 14, "  1인용은 벽뚫기가 가능하며 자신이 얼마나 커졌는지 비교대상이 출력됩니다!!");
    gotoxy(MAP_X + (MAP_WIDTH / 2) - 14, MAP_Y + 15, "                                                      ");
    gotoxy(MAP_X + (MAP_WIDTH / 2) - 14, MAP_Y + 16, "    2인용은 멀티플레이 기반으로 상대와 싸울수 있습니다!");
    gotoxy(MAP_X + (MAP_WIDTH / 2) - 14, MAP_Y + 17, "                                                      ");
    gotoxy(MAP_X + (MAP_WIDTH / 2) - 14, MAP_Y + 18, "          자, 이제 게임모드를 선택해주세요!!");


    while (1) {
        if (kbhit()) { //키입력받음 
            key = getch();
            if (key == ESC) exit(0); // ESC키면 종료 
            else break; //아니면 그냥 계속 진행 
        }
        CursorView(0);
        gotoxy(MAP_X + (MAP_WIDTH / 2) - 7, MAP_Y + 22, " < PRESS ANY KEY TO START > ");
    }
    reset(); // 게임을 초기화
}

void countdown_1(void) {
    int i, j;

    while (kbhit()) getch(); //버퍼에 있는 키값을 버림 

    drawMap();    //맵 테두리를 그림 
    for (i = MAP_Y + 1; i < MAP_Y + MAP_HEIGHT - 1; i++) { // 맵 테두리 안쪽을 빈칸으로 채움 
        for (j = MAP_X + 1; j < MAP_X + MAP_WIDTH - 1; j++)
            gotoxy(j, i, "  ");
    }
    SetConsoleTextAttribute(GetStdHandle(STD_OUTPUT_HANDLE), RED);
    gotoxy(MAP_X + (MAP_WIDTH / 2) - 7, MAP_Y + 14, "3333333333333333333333333333");
    Sleep(400);
    gotoxy(MAP_X + (MAP_WIDTH / 2) - 7, MAP_Y + 14, "                            ");
    Sleep(400);
    gotoxy(MAP_X + (MAP_WIDTH / 2) - 7, MAP_Y + 14, "2222222222222222222222222222");
    Sleep(400);
    gotoxy(MAP_X + (MAP_WIDTH / 2) - 7, MAP_Y + 14, "                            ");
    Sleep(400);
    gotoxy(MAP_X + (MAP_WIDTH / 2) - 7, MAP_Y + 14, "1111111111111111111111111111");
    Sleep(400);
    gotoxy(MAP_X + (MAP_WIDTH / 2) - 7, MAP_Y + 14, "                            ");
    Sleep(400);
    gotoxy(MAP_X + (MAP_WIDTH / 2) - 7, MAP_Y + 14, "staaaaaaart!!!!!!!!!!!!!!!!!");
    Sleep(400);
    reset_1(); // 게임을 초기화 
}

void countdown(void) {
    int i, j;

    while (kbhit()) getch(); //버퍼에 있는 키값을 버림 

    drawMap();    //맵 테두리를 그림 
    for (i = MAP_Y + 1; i < MAP_Y + MAP_HEIGHT - 1; i++) { // 맵 테두리 안쪽을 빈칸으로 채움 
        for (j = MAP_X + 1; j < MAP_X + MAP_WIDTH - 1; j++)
            gotoxy(j, i, "  ");
    }
    SetConsoleTextAttribute(GetStdHandle(STD_OUTPUT_HANDLE), RED);
    gotoxy(MAP_X + (MAP_WIDTH / 2) - 7, MAP_Y + 14, "3333333333333333333333333333");
    Sleep(400);
    gotoxy(MAP_X + (MAP_WIDTH / 2) - 7, MAP_Y + 14, "                            ");
    Sleep(400);
    gotoxy(MAP_X + (MAP_WIDTH / 2) - 7, MAP_Y + 14, "2222222222222222222222222222");
    Sleep(400);
    gotoxy(MAP_X + (MAP_WIDTH / 2) - 7, MAP_Y + 14, "                            ");
    Sleep(400);
    gotoxy(MAP_X + (MAP_WIDTH / 2) - 7, MAP_Y + 14, "1111111111111111111111111111");
    Sleep(400);
    gotoxy(MAP_X + (MAP_WIDTH / 2) - 7, MAP_Y + 14, "                            ");
    Sleep(400);
    gotoxy(MAP_X + (MAP_WIDTH / 2) - 7, MAP_Y + 14, "staaaaaaart!!!!!!!!!!!!!!!!!");
    Sleep(400);
    reset(); // 게임을 초기화 
}

void chooseMod(void) {
    int i, j;

    while (kbhit()) getch(); //버퍼에 있는 키값을 버림 

    drawMap();    //맵 테두리를 그림 
    for (i = MAP_Y + 1; i < MAP_Y + MAP_HEIGHT - 1; i++) { // 맵 테두리 안쪽을 빈칸으로 채움 
        for (j = MAP_X + 1; j < MAP_X + MAP_WIDTH - 1; j++) gotoxy(j, i, "  ");
    }
    SetConsoleTextAttribute(GetStdHandle(STD_OUTPUT_HANDLE), RED);
    gotoxy(MAP_X + (MAP_WIDTH / 2) - 11, MAP_Y + 8,  "        혼자 플레이 하고싶으시면 1을,");
    gotoxy(MAP_X + (MAP_WIDTH / 2) - 11, MAP_Y + 9,  "                            ");
    gotoxy(MAP_X + (MAP_WIDTH / 2) - 11, MAP_Y + 10, "친구와 함께 플레이하고 싶으시면 2를 눌러주세요!!");
    gotoxy(MAP_X + (MAP_WIDTH / 2) - 6, MAP_Y + 20, "< PRESS KEY 1 or 2!!! > ");

    while (1) {
        if (kbhit()) { //키입력받음 
            key = getch();
            if (key == one) {
                keyScan1(); // 1을 입력했을때 1인용 게임 규칙 설명
                break;
            }
            else if (key == two){
                keyScan2();
                break;
            }
            else if (key == ESC)
                exit(0); // ECS를 누르면 종료
            else
                continue;// 아니면 화면 유지
        }
        CursorView(0);
    }
    reset(); // 게임을 초기화  

}



void reset(void) {
    int i;
    system("cls"); // 화면을 지움
    drawMap(); // 맵 테두리를 그림
    while (kbhit()) getch(); // 버퍼에 있는 키값을 버림
    SetConsoleTextAttribute(GetStdHandle(STD_OUTPUT_HANDLE), DarkYellow); // 입력되는 텍스트의 색깔
    dir1 = LEFT; // 첫 번째 뱀의 방향 초기화 (시작시 방향 == LEFT)
    dir2 = LEFT2; // 두 번째 뱀의 방향 초기화 (시작시 방향 == LEFT2)
    speed = 110; // 속도 초기화
    length[0] = 5; // 첫 번째 뱀 길이 초기화
    length[1] = 5; // 두 번째 뱀 길이 초기화

    // 첫 번째 뱀 초기화
    for (i = 0; i < length[0]; i++) {
        x[0][i] = MAP_WIDTH / 2 + i;
        y[0][i] = MAP_HEIGHT / 2;
        gotoxy(MAP_X + x[0][i], MAP_Y + y[0][i], "●");
    }

    // 두 번째 뱀 초기화
    int randX, randY;
   
        randX = rand() % (MAP_WIDTH - length[1]);
        randY = rand() % MAP_HEIGHT;
    for (i = 0; i < length[1]; i++) {
        x[1][i] = randX + i;
        y[1][i] = randY;
        gotoxy(MAP_X + x[1][i], MAP_Y + y[1][i], "●");
    }

    gotoxy(MAP_X + x[0][0], MAP_Y + y[0][0], "⊙"); // 첫 번째 뱀 머리 그림
    gotoxy(MAP_X + x[1][0], MAP_Y + y[1][0], "⊙"); // 두 번째 뱀 머리 그림
    food(); // 음식 생성
    drawMap(); // 맵 테두리를 그림
}

// 주어진 위치에 뱀이나 음식이 있는지 확인하는 함수

void reset_1(void) {
        int i;
        system("cls"); //화면을 지움 
        drawMap(); //맵 테두리를 그림 
        while (kbhit()) getch(); //버퍼에 있는 키값을 버림 
        SetConsoleTextAttribute(GetStdHandle(STD_OUTPUT_HANDLE), DarkYellow);   // 입력되는 텍스트의 색깔
        dir1 = LEFT; // 방향 초기화  (시작시 방향 == LEFT)
        speed = 130; // 속도 초기화 
        length[0] = 5; //뱀 길이 초기화 
        score = 0;
        for (i = 0; i < length[0]; i++) { //뱀 몸통값 입력 
            x[0][i] = MAP_WIDTH / 2 + i;
            y[0][i] = MAP_HEIGHT / 2;
            gotoxy(MAP_X + x[0][i], MAP_Y + y[0][i], "●");
        }
        gotoxy(MAP_X + x[0][0], MAP_Y + y[0][0], "⊙"); //뱀 머리 그림 
        food(); // food 생성  
}


void drawMap(void) {
    int i;
    SetConsoleTextAttribute(GetStdHandle(STD_OUTPUT_HANDLE), BLUE); // 입력되는 텍스트의 색깔을 파란색으로 변경
    for (i = 0; i < MAP_WIDTH; i++) {
        gotoxy(MAP_X + i, MAP_Y, "■");
    }
    for (i = MAP_Y + 1; i < MAP_Y + MAP_HEIGHT - 1; i++) {
        gotoxy(MAP_X, i, "■");
        gotoxy(MAP_X + MAP_WIDTH - 1, i, "■");
    }
    for (i = 0; i < MAP_WIDTH; i++) {
        gotoxy(MAP_X + i, MAP_Y + MAP_HEIGHT - 1, "■");
    }
}

void drawMapTest(void) {
        int i;
        SetConsoleTextAttribute(GetStdHandle(STD_OUTPUT_HANDLE), BLUE);
        for (i = 0; i < MAP_WIDTH; i++) {
            gotoxy(MAP_X + i, MAP_Y, "■");
        }

        for (i = MAP_Y + 1; i < MAP_Y + MAP_HEIGHT - 1; i++) {
            gotoxy(MAP_X, i, "■");
            gotoxy(MAP_X + MAP_WIDTH - 1, i, "■");
        }


        for (i = 0; i < MAP_WIDTH; i++) {
            gotoxy(MAP_X + i, MAP_Y + MAP_HEIGHT - 1, "■");
        }


        printf("\n\n\n\n");
        switch (score / 10) {
        case 1:
        case 2:
        case 3:
        case 4:
        case 5:
        case 6:
        case 7:
        case 8:
        case 9:
            printf("      \033[38;2;116;255;85m\033[48;2;116;255;85m●●●●●●●●●●●●●●\033[0m");
        }

        //int 사람[30][32] = {
        //    {},
        //    {},
        //    {},
        //    {},
        //    {},
        //    {},
        //    {},
        //    {},
        //    {},
        //    {},
        //    {},
        //    {},
        //    {},
        //    {},
        //    {},
        //    {},
        //    {},
        //    {},
        //    {},
        //    {},
        //    {},
        //    {},
        //    {},
        //    {},
        //    {},
        //    {},
        //    {},
        //    {},
        //    {},
        //    {},
        //}

        int 단데기[30][32] = { //1 : 검정 2 : 초록 3 : 흰색
        {3,3,3,3,3,3,1,1,1,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3},
        {3,3,3,3,3,1,1,2,1,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3},
        {3,3,3,3,1,1,2,2,1,1,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3},
        {3,3,3,1,1,2,2,2,2,1,1,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3},
        {3,3,3,1,2,2,2,2,2,2,1,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3},
        {3,3,3,1,2,2,2,2,2,2,1,1,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3},
        {3,3,1,1,2,2,1,1,1,1,1,1,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3},
        {3,3,1,2,2,2,1,2,2,2,1,1,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3},
        {3,3,1,2,2,2,1,3,1,3,1,1,1,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3},
        {3,1,1,1,2,2,1,3,3,3,1,2,1,1,1,1,1,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3},
        {3,1,2,1,2,2,1,1,1,1,1,2,2,2,2,2,1,1,1,3,3,3,3,3,3,3,3,3,3,3,3,3},
        {3,1,2,1,2,2,1,2,2,2,2,2,2,2,2,2,2,2,1,3,3,3,3,3,3,3,3,3,3,3,3,3},
        {1,1,1,1,1,1,1,2,2,2,2,2,2,2,2,2,2,1,1,3,3,3,3,3,3,3,3,3,3,3,3,3},
        {1,2,2,2,2,1,1,2,2,2,2,2,2,2,2,2,1,1,3,3,3,3,3,3,3,3,3,3,3,3,3,3},
        {1,2,2,2,1,1,1,2,2,2,2,2,2,2,2,1,1,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3},
        {1,2,2,2,1,2,1,1,2,2,2,2,2,2,2,1,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3},
        {1,2,1,1,1,2,2,1,2,2,2,2,2,2,2,1,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3},
        {1,1,2,2,2,1,1,1,2,2,2,2,2,2,2,1,1,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3},
        {3,1,1,1,2,2,1,2,2,2,2,2,2,2,2,2,1,1,1,3,3,3,3,3,3,3,3,3,3,3,3,3},
        {3,3,3,1,1,1,1,1,2,2,2,2,2,2,2,2,1,2,1,1,3,3,3,3,3,3,3,3,3,3,3,3},
        {3,3,3,1,1,2,2,1,1,2,2,2,2,2,2,1,2,2,2,1,1,3,3,3,3,3,3,3,3,3,3,3},
        {3,3,3,3,1,1,2,2,1,1,2,2,2,2,2,1,2,2,2,1,1,1,3,3,3,3,3,3,3,3,3,3},
        {3,3,3,3,3,1,2,2,2,1,1,2,2,2,1,1,2,2,1,1,2,1,3,3,3,3,3,3,3,3,3,3},
        {3,3,3,3,3,1,1,2,2,2,1,1,2,2,1,2,2,2,1,2,2,1,1,3,3,3,3,3,3,3,3,3},
        {3,3,3,3,3,3,1,1,2,2,2,1,1,1,1,2,2,2,1,2,2,2,1,1,1,3,3,3,3,3,3,3},
        {3,3,3,3,3,3,3,1,1,1,2,2,2,1,2,2,2,1,1,2,2,1,1,2,1,1,1,3,3,3,3,3},
        {3,3,3,3,3,3,3,3,3,1,1,1,1,1,2,2,2,1,2,2,2,1,2,1,1,2,1,3,3,3,3,3},
        {3,3,3,3,3,3,3,3,3,3,3,3,3,1,1,1,1,1,2,2,2,1,2,1,2,1,1,3,3,3,3,3},
        {3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,1,1,1,1,1,1,1,1,1,1,3,3,3,3,3,3},
        {0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0}
        };
        for (int i = 0; i < 30; i++) {
            gotoxy2(130, i + 3);
            for (int j = 0; j < 32; j++) {
                if (단데기[i][j] == 0) {
                    printf("  ");
                }
                else if (단데기[i][j] == 1) {
                    printf("\033[38;2;0;0;0m\033[48;2;0;0;0m■\033[0m");
                }
                else if (단데기[i][j] == 2) {
                    printf("\033[38;2;116;255;85m\033[48;2;116;255;85m■\033[0m");
                }
                else if (단데기[i][j] == 3) {
                    printf("\033[38;2;255;255;255m\033[48;2;255;255;255m■\033[0m");
                }
            }
            printf("\n");
        } //단데기 도트 출력

}

void food(void) {
    int i;

    int food_crush_on = 0;//food가 뱀 몸통좌표에 생길 경우 on 
    int r = 0; //난수 생성에 사동되는 변수 

    while (1) {
        food_crush_on = 0;
        srand((unsigned)time(NULL) + r); //난수표생성 
        food_x = (rand() % (MAP_WIDTH - 2)) + 1;    //난수를 좌표값에 넣음 
        food_y = (rand() % (MAP_HEIGHT - 2)) + 1;

        for (i = 0; i < length[0]; i++) { //food가 첫번째 플레이어의 뱀 몸통과 겹치는지 확인  
            if (food_x == x[0][i] && food_y == y[0][i]) {
                food_crush_on = 1; //겹치면 food_crush_on 를 on 
                r++;
                break;
            }
        }

        for (i = 0; i < length[1]; i++) { //food가 두 번째 플레이어의 뱀 몸통과 겹치는지 확인  
            if (food_x == x[1][i] && food_y == y[1][i]) {
                food_crush_on = 1; //겹치면 food_crush_on 를 on 
                r++;
                break;
            }
        }

        if (food_crush_on == 1) continue; //겹쳤을 경우 while문을 다시 시작 
        SetConsoleTextAttribute(GetStdHandle(STD_OUTPUT_HANDLE), RED);
        gotoxy(MAP_X + food_x, MAP_Y + food_y, "♥"); //안겹쳤을 경우 좌표값에 food를 찍고 
        speed -= 2; //속도 증가 
        break;

    }
}

void gameOver(void) {
    reset();
    SetConsoleTextAttribute(GetStdHandle(STD_OUTPUT_HANDLE), DarkYellow);
    gotoxy(MAP_X + (MAP_WIDTH / 2) - 6, MAP_Y + 10, "+----------------------+");
    gotoxy(MAP_X + (MAP_WIDTH / 2) - 6, MAP_Y + 11, "|      !!1번사망!!     |");
    gotoxy(MAP_X + (MAP_WIDTH / 2) - 6, MAP_Y + 12, "|      !!1번사망!!     |");
    gotoxy(MAP_X + (MAP_WIDTH / 2) - 6, MAP_Y + 13, "|      !!1번사망!!     |");
    gotoxy(MAP_X + (MAP_WIDTH / 2) - 6, MAP_Y + 14, "|      !!1번사망!!     |");
    gotoxy(MAP_X + (MAP_WIDTH / 2) - 6, MAP_Y + 15, "+----------------------+");
    gotoxy(MAP_X + (MAP_WIDTH / 2) - 15, MAP_Y + 20, "모드를 선택하세요, 1인용 모드를 선택하시기 전엔 F11을 눌러주세요!! ");
    Sleep(500);
    while (1) {
        if (kbhit()) { //키입력받음 
            key = getch();
            if (key == ESC)
                exit(0); // ESC키면 종료 
            else if (key == one)
                keyScan1();
            else if (key == two)
                keyScan2();
        }
        CursorView(0);
    }
}

void gameOver2(void) {
    reset();
    SetConsoleTextAttribute(GetStdHandle(STD_OUTPUT_HANDLE), DarkYellow);
    gotoxy(MAP_X + (MAP_WIDTH / 2) - 6, MAP_Y + 10, "+----------------------+");
    gotoxy(MAP_X + (MAP_WIDTH / 2) - 6, MAP_Y + 11, "|      !!2번사망!!     |");
    gotoxy(MAP_X + (MAP_WIDTH / 2) - 6, MAP_Y + 12, "|      !!2번사망!!     |");
    gotoxy(MAP_X + (MAP_WIDTH / 2) - 6, MAP_Y + 13, "|      !!2번사망!!     |");
    gotoxy(MAP_X + (MAP_WIDTH / 2) - 6, MAP_Y + 14, "|      !!2번사망!!     |");
    gotoxy(MAP_X + (MAP_WIDTH / 2) - 6, MAP_Y + 15, "+----------------------+");
    gotoxy(MAP_X + (MAP_WIDTH / 2) -15, MAP_Y + 20, "모드를 선택하세요, 1인용 모드를 선택하시기 전엔 F11을 눌러주세요!! ");

    Sleep(500);
    while (1) {
        if (kbhit()) { //키입력받음 
            key = getch();
            if (key == ESC)
                exit(0); // ESC키면 종료 
            else if (key == one)
                keyScan1();
            else if (key == two)
                keyScan2();
        }
        CursorView(0);
    }
}



void cursor(int n) { //뱀 이동시 커서 숨기는 함수
    HANDLE hConsole;
    CONSOLE_CURSOR_INFO ConsoleCursor;

    hConsole = GetStdHandle(STD_OUTPUT_HANDLE);
    ConsoleCursor.bVisible = n;
    ConsoleCursor.dwSize = 1;

    SetConsoleCursorInfo(hConsole, &ConsoleCursor);
}
```
