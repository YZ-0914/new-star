# new-star
#define _CRT_SECURE_NO_WARNINGS 1

#include<graphics.h>
#include<time.h>
#include<stdio.h>
#include<math.h>
#include<stdlib.h>
#include<windows.h>
#pragma comment(lib, "winmm.lib")
#define NUM 10
#define PI 3.1415925

//烟花弹
struct jet
{
	int x, y;
	int hx, hy;
	bool shoot;
	DWORD t1, t2, dt;
	IMAGE img[2];
	byte n : 1;
}jet[NUM];

//烟花
struct Fire
{
	int x, y;
	int r;
	int max_r;
	int cen_x, cen_y;
	int width, height;
	int xy[240][240];
	bool draw;
	bool show;
	DWORD t1, t2, dt;
}fire[NUM];

//初始化函数
void FireInit(int i)
{
	//初始化烟花弹
	jet[i].t1 = GetTickCount();
	jet[i].shoot = false;
	jet[i].dt = 10;
	jet[i].n = 0;
	fire[i].r = 0;
	fire[i].dt = 5;
	fire[i].t1 = GetTickCount();
	fire[i].max_r = rand() % 50 + 100;
	fire[i].cen_x = rand() % 30 + 80;
	fire[i].cen_y = rand() % 30 + 80;
	fire[i].width = 240;
	fire[i].height = 240;
}

//加载
void Load()
{
	//加载烟花弹
	IMAGE jetimg;
	loadimage(&jetimg, "./res/launch.jpg", 200, 50);
	SetWorkingImage(&jetimg);
	for (int i = 0; i < NUM; i++)
	{
		int n = rand() % 5;
		getimage(&jet[i].img[0], n * 20, 0, 20, 50);
		getimage(&jet[i].img[1], (n + 5) * 20, 0, 20, 50);
	}
	SetWorkingImage(NULL);
	//加载烟花
	IMAGE fireimage, Fireimage;
	loadimage(&Fireimage, "./res/flower.jpg", 3120, 240);
	for (int i = 0; i < NUM; i++)
	{
		SetWorkingImage(&Fireimage);
		getimage(&fireimage, i * 240, 0, 240, 240);
		SetWorkingImage(&fireimage);
		for (int a = 0; a < 240; a++)
		{
			for (int b = 0; b < 240; b++)
			{
				fire[i].xy[a][b] = getpixel(a, b);
			}
		}

	}
	SetWorkingImage(NULL);
}

//选择烟花弹
void ChoiceJet(DWORD& t1)
{
	DWORD t2 = GetTickCount();
	if (t2 - t1 > 100)
	{
		//烟花弹个数
		int i = rand() % 10;
		//不处于发射状态
		if (jet[i].shoot == false && fire[i].show == false)
		{
			//烟花弹
			jet[i].x = rand() % 1000;
			jet[i].y = rand() % 100 + 450;
			jet[i].hx = jet[i].x;
			jet[i].hy = rand() % 300;
			jet[i].shoot = true;

			putimage(jet[i].x, jet[i].y, &jet[i].img[jet[i].n], SRCINVERT);
		}
		t1 = t2;
	}
}

//判断发射
void Shoot()
{
	for (int i = 0; i < NUM; i++)
	{
		jet[i].t2 = GetTickCount();
		if (jet[i].t2 - jet[i].t1 >= jet[i].dt&&jet[i].shoot == true)
		{
			putimage(jet[i].x, jet[i].y, &jet[i].img[jet[i].n], SRCINVERT);
			if (jet[i].y >= jet[i].hy)
			{
				jet[i].n++;
				jet[i].y -= 5;
			}
			putimage(jet[i].x, jet[i].y, &jet[i].img[jet[i].n], SRCINVERT);
			if (jet[i].y <= jet[i].hy)
			{
				putimage(jet[i].x, jet[i].y, &jet[i].img[jet[i].n], SRCINVERT);
				jet[i].shoot = false;
				//达到最大高度，接下来交给烟花
				//重新发射
				fire[i].x = jet[i].hx;
				fire[i].y = jet[i].hy;
				fire[i].show = true;
			}
		}
		jet[i].t1 = jet[i].t2;
	}
}

//显示烟花
void ShowFire(DWORD* pMem)
{
	int drt[16] = { 5, 5, 5, 5, 5, 10, 25, 25, 25, 25, 55, 55, 55, 55, 55, 65 };
	for (int i = 0; i < NUM; i++)
	{
		fire[i].t2 = GetTickCount();
		if (fire[i].t2 - fire[i].t1 >= fire[i].dt&&fire[i].show == true)
		{
			if (fire[i].r < fire[i].max_r)
			{
				fire[i].r++;
				fire[i].dt = drt[fire[i].r / 10];
				fire[i].draw = true;
			}
			if (fire[i].r >= fire[i].max_r)
			{
				fire[i].draw = false;
				FireInit(i);
			}
			fire[i].t1 = fire[i].t2;
			//如果该号炮花可爆炸，根据当前爆炸半径画烟花，颜色接近黑色的不输出。
			if (fire[i].draw)
			{
				for (double a = 0; a <= 6.28; a += 0.01)
				{
					int x1 = (int)(fire[i].cen_x + fire[i].r*cos(a));
					int y1 = (int)(fire[i].cen_y - fire[i].r*sin(a));
					if (x1>0 && x1 < fire[i].width&&y1>0 && y1 < fire[i].height)
					{
						int b = fire[i].xy[x1][y1] & 0xff;
						int g = (fire[i].xy[x1][y1] >> 8) & 0xff;
						int r = (fire[i].xy[x1][y1] >> 16);
						//烟花像素点在窗口上的坐标
						int xx = (int)(fire[i].x + fire[i].r*cos(a));
						int yy = (int)(fire[i].y - fire[i].r*sin(a));
						//较暗的像素点不输出、防止越界
						if (r>0x20 && g > 0x20 && b > 0x20 && xx > 0 && xx < 1000 && yy>0 && yy < 600)
						{
							pMem[yy * 1000 + xx] = BGR(fire[i].xy[x1][y1]);
						}
						fire[i].draw = false;
					}
				}
			}
		}
	}
}

//菜单界面
void welcome()
{
	setcolor(YELLOW);
	for (int i = 0; i < 50; i++)
	{
		int x = 600 + int(180 * sin(PI * 2 * i / 60));
		int y = 200 + int(180 * cos(PI * 2 * i / 60));
		cleardevice();
		settextstyle(i, 0, "楷体");
		outtextxy(x - 80, y, "一首好听的歌");
		outtextxy(x - 10, y+100, "俊杰的歌哦！");
		Sleep(65);
	}
	Sleep(130);
	cleardevice();
	settextstyle(25, 0,"楷体");
	outtextxy(400, 200, "你的心有一道墙");
	outtextxy(400, 250, "但我发现一扇窗");
	outtextxy(400, 300, "偶尔透出一丝温暖的微光");
	outtextxy(400, 350, "就算你有一道墙");
	outtextxy(400, 400, "我的爱会攀上窗台盛放");
	outtextxy(400, 450, "打开窗你会看到悲伤融化");
	outtextxy(650, 500, "心墙");
	Sleep(2000);
	cleardevice();
}

//主函数
int main()
{
	//初始界面（1000，600）
	initgraph(1000, 600);
	//初始化种子
	srand((unsigned int)time(NULL));
	//音乐
	mciSendString("open ./res/心墙.mp3 alias music", 0, 0, 0);
	mciSendString("play music", 0, 0, 0);
	welcome();
	DWORD t1 = GetTickCount();
	DWORD* pMem = GetImageBuffer();
	for (int i = 0; i < NUM; i++)
	{
		FireInit(i);
	}
	Load();
	BeginBatchDraw();
	while (1)
	{
		//随机选择像素点擦除
		for (int clr = 0; clr < 200; clr++)
		{
			int px1 = rand() % 1000;
			int py1 = rand() % 600;
			//防止越界
			if (py1 < 599)
			{
				//对显存赋值擦除像素点
				pMem[py1 * 1000 + px1] = pMem[py1 * 1000 + px1 + 1] = BLACK;
			}
		}
		ChoiceJet(t1);
		Shoot();
		ShowFire(pMem);
		FlushBatchDraw();
	}
	system("pause");
	return 0;
}
