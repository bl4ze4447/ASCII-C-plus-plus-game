# ASCII-C-plus-plus-game
ASCII Game with a map editor/creator in C++


The source file contains an ASCII game that has collision checking, jump checking, physics, a map creator/editor to make your own maps and it comes with a test map to test the app.

THIS DOESNT WORK ON LINUX YET // COMING SOON


>![My image](ignore.png)


>![C++](https://img.shields.io/badge/c++-%2300599C.svg?style=for-the-badge&logo=c%2B%2B&logoColor=white)
```
#include <iostream>
#include <string>
#include <conio.h>
#include <thread>
#include <chrono>
#include <limits>
using std::cout;
using std::cin;
using std::getline;
using std::string;
using std::thread;
using std::chrono::milliseconds;
using std::numeric_limits;
using std::streamsize;
// using std::this_thread::sleep_for; -> Commented out because it makes code harder to understand / just sleep_for() could mean anything without hte this_thread
constexpr char KEY_UP = 'w';
constexpr char KEY_LEFT = 'a';
constexpr char KEY_RIGHT = 'd';
string map[] = { "                                     ",
                "                                     ",
                "                                     ",
                "                                     ",
                "                                     ",
                "              ---                    ",
                "           ---- -              ------",
                "---------  -    -----     ---  -    -",
                "        -  -----------  --- -  -    -",
                "---------xx-         -xx-   -xx-    -"
};

// Functions
void isGrounded(struct Player* player);
void display(struct Player player);
void inputHandler(struct Player* player);
void jumpCorrection(struct Player* player);
void jumpEnded(struct Player* player);
void resetCin();
void mainCreator();

// Player
struct Player
{
    unsigned short int X_Axis, Y_Axis;
    bool isGrounded;
    bool jumped;
    unsigned char character;
};

// THREAD
thread jumpEnder;
thread correctJump;

int main()
{
    // INITIALIZING
    Player player{};
    player.isGrounded = false;
    player.jumped = false;
    player.X_Axis = 0; // When changing the map make sure these values are set inside the 
    player.Y_Axis = 6; // maps size or you will get a segmentation fault
    player.character = 'O';
    while (true) {
        unsigned short int c{ 0 };
        cout << "PLAY / Map Editor(1/2): "; cin >> c;
        if (c == 1) {
            break;
        }
        else if (c == 2) {
            mainCreator();
        }
        else {
            resetCin();
        }
    }
    while (true)
    {
        display(player);
        isGrounded(&player);
        inputHandler(&player);
        isGrounded(&player);
        correctJump = thread(jumpCorrection, &player);
        correctJump.detach();
    }
    mainCreator();
}


// Ground/Death check
void isGrounded(struct Player* player)
{
    if (map[player->Y_Axis + 1][player->X_Axis] == '-')
    {
        player->isGrounded = true;
        player->jumped = false;
        return;
    }
    else if (map[player->Y_Axis + 1][player->X_Axis] == 'x')
    {
        cout << "> GAME OVER";
        exit(0);
    }
    player->isGrounded = false;
}

// Display
void display(struct Player player)
{
    cout << "\x1B[2J\x1B[H";
    for (size_t Y_AXIS = 0; Y_AXIS < sizeof(map) / sizeof(string); Y_AXIS++)
    {
        for (size_t X_AXIS = 0; X_AXIS < map[Y_AXIS].length(); X_AXIS++)
        {
            if (X_AXIS == player.X_Axis && Y_AXIS == player.Y_Axis)
            {
                map[Y_AXIS][X_AXIS] = player.character;
            }
            else if (map[Y_AXIS][X_AXIS] == player.character) {
                map[Y_AXIS][X_AXIS] = ' ';
            }
            cout << map[Y_AXIS][X_AXIS];
        }
        cout << "\n";
    }
    cout << player.isGrounded << '\n';
}

// Handle input from user to move the position of the player
void inputHandler(struct Player* player)
{
    char input = _getch();
    switch (input)
    {
    case KEY_UP:
        if (player->isGrounded)
        {
            jumpEnder = thread(jumpEnded, player);
            jumpEnder.detach();
        }
        break;
    case KEY_LEFT:
        if (map[player->Y_Axis][player->X_Axis - 1] == ' ')
        {
            player->X_Axis -= 1;
        }
        break;
    case KEY_RIGHT:
        if (map[player->Y_Axis][player->X_Axis + 1] == ' ')
        {
            player->X_Axis += 1;
        }
        break;
    default:
        resetCin();
    }
}

// Make character not float in the air and actually go down
void jumpCorrection(struct Player* player)
{
    std::this_thread::sleep_for(milliseconds(100));
    while (map[player->Y_Axis + 1][player->X_Axis] == ' ' && !player->jumped)
    {
        player->Y_Axis += 1;
        std::this_thread::sleep_for(milliseconds(150));
        display(*player);
        isGrounded(player);
    }
}

// Check if the jump has ended but also handle the jump
void jumpEnded(struct Player* player)
{
    if (map[player->Y_Axis - 1][player->X_Axis] != ' ') {
        return;
    }
    player->jumped = true;
    player->Y_Axis -= 1;
    display(*player);
    if (map[player->Y_Axis - 1][player->X_Axis] != ' ') {
        player->jumped = false;
        display(*player);
        return;
    }
    std::this_thread::sleep_for(milliseconds(200));
    player->Y_Axis -= 1;
    display(*player);
    player->jumped = false;
}

void mainCreator() {
    unsigned short int rows, columns;
    cout << "\tLimits: ROWS: 20 / COLUMNS: 50 per frame\n";
    cout << "Nummber of rows: "; cin >> rows;
    if (rows > 20) { mainCreator(); }
    if (cin.fail()) {
        resetCin();
        mainCreator();
    }
    cout << "Number of columns: "; cin >> columns;
    if (columns > 50) { mainCreator(); }
    if (cin.fail()) {
        resetCin();
        mainCreator();
    }
    cout << "Creating map editor...\n";
    unsigned char** map = new unsigned char* [rows];
    for (int i = 0; i < rows; i++) {
        map[i] = new unsigned char[columns];
        for (int j = 0; j < columns; j++) {
            map[i][j] = 'x';
        }
        cout << '\n';
    }
    cout << "Map created, starting editor\n";
    bool finished = false;
    unsigned short int X_AXIS{ 1 }, Y_AXIS{ 1 };
    while (!finished) {
        cout << "\x1B[2J\x1B[H";
        for (int i = 0; i < rows; i++) {
            for (int j = 0; j < columns; j++) {
                cout << map[i][j];
            }
            cout << '\n';
        }
        cout << "EDITOR POSITION: (inverted)Y: " << Y_AXIS << " / X: " << X_AXIS << '\n';
        cout << "CONTROLS: (W/S): switch ascii character / (L-Left ; D-Down ; U-Up ; R-Right): move between characters\n";
        cout << "          (Z + W/S): switch entire line / (P): Print string to use\n";
        char input = _getch();
        switch (input) {
        case 'l':
            if (X_AXIS > 1) {
                X_AXIS--;
            }
            break;
        case 'd':
            if (Y_AXIS < rows) {
                Y_AXIS++;
            }
            break;
        case 'u':
            if (Y_AXIS > 1) {
                Y_AXIS--;
            }
            break;
        case 'r':
            if (X_AXIS < columns) {
                X_AXIS++;
            }
            break;
        case 'w':
            switch (map[Y_AXIS - 1][X_AXIS - 1]) {
            case 'x':
                map[Y_AXIS - 1][X_AXIS - 1] = '-';
                break;
            case '-':
                map[Y_AXIS - 1][X_AXIS - 1] = ' ';
                break;
            case ' ':
                map[Y_AXIS - 1][X_AXIS - 1] = 'x';
                break;
            }
            break;
        case 's':
            switch (map[Y_AXIS - 1][X_AXIS - 1]) {
            case 'x':
                map[Y_AXIS - 1][X_AXIS - 1] = ' ';
                break;
            case '-':
                map[Y_AXIS - 1][X_AXIS - 1] = 'x';
                break;
            case ' ':
                map[Y_AXIS - 1][X_AXIS - 1] = '-';
                break;
            }
            break;
        case 'q':
            finished = true;
            break;
        case 'z':
            unsigned char temp;
            temp = _getch();
            switch (temp) {
            case 'w':
                for (int j = 0; j < columns; j++) {
                    switch (map[Y_AXIS - 1][j]) {
                    case 'x':
                        map[Y_AXIS - 1][j] = '-';
                        break;
                    case '-':
                        map[Y_AXIS - 1][j] = ' ';
                        break;
                    case ' ':
                        map[Y_AXIS - 1][j] = 'x';
                        break;
                    }
                }
                break;
            case 's':
                for (int j = 0; j < columns; j++) {
                    switch (map[Y_AXIS - 1][j]) {
                    case 'x':
                        map[Y_AXIS - 1][j] = ' ';
                        break;
                    case '-':
                        map[Y_AXIS - 1][j] = 'x';
                        break;
                    case ' ':
                        map[Y_AXIS - 1][j] = '-';
                        break;
                    }
                }
                break;
            default:
                resetCin();
            }
            break;
        case 'p':
            finished = true;
            cout << "string map[] = {";
            for (int i = 0; i < rows; i++) {
                cout << "\"";
                for (int j = 0; j < columns; j++) {
                    cout << map[i][j];
                }
                if (i == rows - 1) {
                    cout << "\"};\n";
                    finished = true;
                    cout << "Press enter to return to main menu..\n";
                    char a{' '};
                    a = _getch();
                    if (a != ' ') {
                        cout << "\x1B[2J\x1B[H";
                        return;
                    }
                    break;
                }
                cout << "\",\n\t";
            }
            break;
        default:
            resetCin();
        }
    }
}

void resetCin() {

    cin.clear();
    cin.ignore(numeric_limits<streamsize>::max(), '\n');
}
```
