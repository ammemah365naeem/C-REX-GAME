# C-REX-GAME
Semester 1 Project


#include "include/config.h"
#include "include/raylib.h"

//things that will remain constant throughout the game
const int SCREEN_WIDTH  = 800;
const int SCREEN_HEIGHT = 450;
const int GROUND_Y      = 380;
const int MAX_METEORS   = 2;


struct Meteor {
    Vector2 position;
    Vector2 speed;
    bool active;
    bool hitGround;
};

struct Projectile {
    Vector2 position;
    float speed;
    float distance;
    bool active;
};

int main()
{
    InitWindow(SCREEN_WIDTH, SCREEN_HEIGHT, "C-Rex w/ a&i");
    SetTargetFPS(40);

    Texture2D dinoTexture   = LoadTexture("dino1.jpg");
    Texture2D meteorTexture = LoadTexture("meteor.png");
    Texture2D cloudTexture  = LoadTexture("cloud.png");
    Texture2D moonTexture   = LoadTexture("moon.png");

    float dinoScale   = 0.45f;//resizing of original picture, 
    float meteorScale = 0.12f;
    float cloudScale  = 0.10f;
    float moonScale   = 0.22f;

    float dinoW   = dinoTexture.width * dinoScale;
    float dinoH   = dinoTexture.height * dinoScale;
    float meteorW = meteorTexture.width * meteorScale;
    float meteorH = meteorTexture.height * meteorScale;

    Vector2 dinoPos = {150, GROUND_Y - dinoH};
    float dinoVelY = 0;
    float gravity = 800;
    float jumpPower = -620;
    float dinoForwardSpeed = 80;

    
    struct Projectile bullet = {0};
    float bulletSpeed = 600;
    float bulletRange = 650;

//array can hold only 2 max meteors at a time
    struct Meteor meteors[MAX_METEORS] = {};
    float spawnTimer = 0;

    float worldSpeed = 120;
    Vector2 cloudPos = {320, 70};
    Vector2 moonOffset = {70, -12};

    int score = 0;
    int level = 1;
    bool gameOver = false;

    while (!WindowShouldClose())
    {
        float dt = GetFrameTime();

        if (!gameOver)
        {
            dinoPos.x += dinoForwardSpeed * dt;
            if (dinoPos.x > 350) dinoPos.x = 350;

            if (IsKeyPressed(KEY_SPACE) &&
                dinoPos.y >= GROUND_Y - dinoH)
            {
                dinoVelY = jumpPower;
            }

            dinoVelY += gravity * dt;
            dinoPos.y += dinoVelY * dt;

            if (dinoPos.y > GROUND_Y - dinoH)
            {
                dinoPos.y = GROUND_Y - dinoH;
                dinoVelY = 0;///
            }

            if (IsKeyPressed(KEY_F) && !bullet.active)
            {
                bullet.active = true;
                bullet.position = (Vector2){
                    dinoPos.x + dinoW,
                    dinoPos.y + dinoH / 2// bullet comes from dino hand
                };
                bullet.distance = 0;
            }
        }


        if (score < 10) level = 1;
        else if (score < 20) level = 2;
        else level = 3;

       //levels
        if (level == 1)
        {
            worldSpeed = 130;
        }
        else if (level == 2)
        {
            worldSpeed = 180;
        }
        else
        {
            worldSpeed = 220;
        }

        cloudPos.x -= worldSpeed * dt * 0.25f;
        if (cloudPos.x < -100)
            cloudPos.x = SCREEN_WIDTH + 100;
//mteors diagonal falling and collision
        spawnTimer += dt;
        int maxActive = (level < 3) ? 1 : 2; // ternary operators used <they are like if-else statements>.

        if (spawnTimer > 1.4f && !gameOver)
        {
            for (int i = 0; i < maxActive; i++)//diagonally falling and checking to generate more meteors.
            {
                if (!meteors[i].active)
                {
                    meteors[i].position = (Vector2){SCREEN_WIDTH + 80, -100};
                    meteors[i].speed = (Vector2){
                        -120 - level * 40,
                        260 + level * 30
                    };
                    meteors[i].active = true;
                    meteors[i].hitGround = false;
                    spawnTimer = 0;
                    break;
                }
            }
        }

        for (int i = 0; i < MAX_METEORS; i++)
        {
            if (!meteors[i].active) continue;

            if (!meteors[i].hitGround)
            {
                meteors[i].position.x += meteors[i].speed.x * dt;// falling
                meteors[i].position.y += meteors[i].speed.y * dt;//

                if (meteors[i].position.y >= GROUND_Y - meteorH)
                {

                    meteors[i].position.y = GROUND_Y - meteorH;//falling down and hitting the ground
                    meteors[i].speed.y = 0;
                    meteors[i].hitGround = true;
                }
            }
            else
            {
                meteors[i].position.x += meteors[i].speed.x * dt;
            }

            Rectangle meteorRect = {
                meteors[i].position.x + 6,
                meteors[i].position.y + 6,
                meteorW - 12,
                meteorH - 12
            };

            Rectangle dinoRect = {
                dinoPos.x + 8,
                dinoPos.y + 10,
                dinoW - 16,
                dinoH - 20
            };

            if (CheckCollisionRecs(dinoRect, meteorRect))
                gameOver = true;

            if (bullet.active)
            {
                Rectangle bulletRect = {
                    bullet.position.x,
                    bullet.position.y,
                    10, 4
                };

                if (CheckCollisionRecs(bulletRect, meteorRect))
                {
                    meteors[i].active = false;
                    bullet.active = false;
                    score++;
                }
            }

            if (meteors[i].position.x < -100)
            {
                meteors[i].active = false;
                score++;
            }
        }

        if (bullet.active)
        {
            bullet.position.x += bulletSpeed * dt;
            bullet.distance += bulletSpeed * dt;
            if (bullet.distance > bulletRange)
                bullet.active = false;
        }

        if (gameOver && IsKeyPressed(KEY_R))
        {
            gameOver = false;
            score = 0;
            level = 1;
            dinoPos = (Vector2){150, GROUND_Y - dinoH};
            dinoVelY = 0;
            bullet.active = false;

            for (int i = 0; i < MAX_METEORS; i++)
                meteors[i].active = false;
        }

        BeginDrawing();
        ClearBackground(WHITE);

        DrawTextureEx(cloudTexture, cloudPos, 0, cloudScale, LIGHTGRAY);
        DrawTextureEx(moonTexture,
            (Vector2){cloudPos.x + moonOffset.x, cloudPos.y + moonOffset.y},
            0, moonScale, WHITE);

        DrawLine(0, GROUND_Y, SCREEN_WIDTH, GROUND_Y, DARKGRAY);
        DrawTextureEx(dinoTexture, dinoPos, 0, dinoScale, WHITE);

        for (int i = 0; i < MAX_METEORS; i++)
            if (meteors[i].active)
                DrawTextureEx(meteorTexture, meteors[i].position, 0, meteorScale, WHITE);

        if (bullet.active)
            DrawRectangle(bullet.position.x, bullet.position.y, 10, 4, BLACK);

        DrawText(TextFormat("Score: %d", score), 20, 15, 20, BLACK);
        DrawText(TextFormat("Level: %d", level), 20, 40, 20, BLACK);
        DrawText("SPACE - Jump   F - Shoot   R - Restart", 330, 15, 18, BLACK);

        if (gameOver)
            DrawText("GAME OVER!", 320, 200, 30, RED);

        EndDrawing();
    }

    CloseWindow();
    return 0;
}
