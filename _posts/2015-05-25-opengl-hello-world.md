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