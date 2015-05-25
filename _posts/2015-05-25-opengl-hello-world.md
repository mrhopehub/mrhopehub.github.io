---
layout: posts
title: "opengl变换"
---
# {{ page.title }}
<xmp class="prettyprint linenums">#include <stdlib.h>
#include <stdio.h>
#include <GL/glut.h>
#define WINDOW_WIDTH 800
#define WINDOW_HEIGTH 640
#define WINDOW_POSITION_X 0
#define WINDOW_POSITION_Y 0
#define CLIP_LEFT	-200
#define CLIP_RIGHT	 200
#define CLIP_BOTTOM -200
#define CLIP_TOP	 200
#define CLIP_ZNEAR	-1.0
#define CLIP_ZFAR	 1.0

void init(int *argc, char *argv[]);
void myReshape(int w, int h);
void myDisplay(void);

int main(int argc, char *argv[])
{
	init(&argc, argv);
	
	glutCreateWindow("第一个OpenGL程序");
	
	glutDisplayFunc(myDisplay);
	glutReshapeFunc(myReshape);
	
	glutMainLoop();
	
	return 0;
}

void init(int *argc, char *argv[])
{
	glutInit(argc, argv);
	
	glutInitDisplayMode(GLUT_RGB | GLUT_SINGLE);
	glutInitWindowSize(WINDOW_WIDTH, WINDOW_HEIGTH);
	glutInitWindowPosition(WINDOW_POSITION_X, WINDOW_POSITION_Y);
	
	glClearColor(0.0, 0.0, 0.0, 0.0);
	glColor3f(1.0, 1.0, 1.0);

}

void myDisplay(void)
{
	glClear(GL_COLOR_BUFFER_BIT);
	{
		glRasterPos2d(-200, -200);
		glutBitmapCharacter(GLUT_BITMAP_9_BY_15,'H');
	}
	glLoadIdentity();
	{
		glBegin(GL_QUADS);
		glVertex2i( 50, 100);
		glVertex2i(200, 100);
		glVertex2i(200, 150);
		glVertex2i( 50, 150);
		glEnd();
	}
	//旋转
	{
		glColor3f(1.0, 0.0, 0.0);
		glRotatef(90, 0, 0, 1.0);
		
		glBegin(GL_QUADS);
		glVertex2i( 50, 100);
		glVertex2i(200, 100);
		glVertex2i(200, 150);
		glVertex2i( 50, 150);
		glEnd();
	}
	//平移
	{
		glColor3f(0.0, 1.0, 0.0);
		glTranslatef(-50, -50, 0);
		
		glBegin(GL_QUADS);
		glVertex2i( 50, 100);
		glVertex2i(200, 100);
		glVertex2i(200, 150);
		glVertex2i( 50, 150);
		glEnd();
	}
	//缩放
	{
		glColor3f(0.0, 0.0, 1.0);
		glScalef(0.5, 0.5, 1.0);
		glBegin(GL_QUADS);
		glVertex2i( 50, 100);
		glVertex2i(200, 100);
		glVertex2i(200, 150);
		glVertex2i( 50, 150);
		glEnd();
	}
	glRasterPos2d(0, 0);
	glutBitmapCharacter(GLUT_BITMAP_9_BY_15,'H');
	glFlush();
}

void myReshape(int w, int h)
{
	//设置视口，窗口的（0， 0）、w、h
	glViewport(0, 0, (GLsizei) w, (GLsizei) h);

	//设置裁剪范围
	glMatrixMode(GL_PROJECTION);
	glLoadIdentity();
	glOrtho(CLIP_LEFT, CLIP_RIGHT, CLIP_BOTTOM, CLIP_TOP, CLIP_ZNEAR, CLIP_ZFAR);

	//设置观察矩阵投影模式
	glMatrixMode(GL_MODELVIEW);
	glLoadIdentity();
}
</xmp>

## 关于glutReshapeFunc(myReshape);
<xmp class="my_xmp_class">若去掉reshape函数，即把reshape中的函数放进init中，则myDisplay并不能正常工作。
但是像下面的初始化，则myDisplay能正常工作。</xmp>
<xmp class="prettyprint linenums">#include "glut.h"
void myDisplay(void)
{
	glClear(GL_COLOR_BUFFER_BIT);
	glRectf(-0.5f, -0.5f, 0.5f, 0.5f);
	glFlush();
}
int main(int argc, char *argv[])
{
	glutInit(&argc, argv);
	glutCreateWindow("第一个OpenGL程序");
	glutDisplayFunc(&myDisplay);
	glutMainLoop();
}</xmp>
## 设置裁剪范围
<xmp class="prettyprint linenums">	glMatrixMode(GL_PROJECTION);
	glLoadIdentity();
	glOrtho(CLIP_LEFT, CLIP_RIGHT, CLIP_BOTTOM, CLIP_TOP, CLIP_ZNEAR, CLIP_ZFAR);</xmp>
<xmp class="my_xmp_class">这是一个整体代码段，单独的glOrtho(CLIP_LEFT, CLIP_RIGHT, CLIP_BOTTOM, CLIP_TOP, CLIP_ZNEAR, CLIP_ZFAR)并没有意义，也不会起作用</xmp>
## 关于glRasterPos2d(0, 0);
<xmp class="my_xmp_class">该函数指定的当前位置，并不是最终的光栅坐标，而是跟glVertex2i类似，是世界坐标系，所以glColor3f(1.0, 0.0, 0.0)等等还会起作用。
但是glutBitmapCharacter并不受变换的影响。</xmp>
## 关于glClear(GL_COLOR_BUFFER_BIT);
<xmp class="my_xmp_class">该程式会清除整个窗口，而不是当前的视口。</xmp>
## 关于glViewport(0, 0, (GLsizei) w, (GLsizei) h);
<xmp class="my_xmp_class">（0,0）是窗口的左下角。而不是世界坐标系的原点。</xmp>