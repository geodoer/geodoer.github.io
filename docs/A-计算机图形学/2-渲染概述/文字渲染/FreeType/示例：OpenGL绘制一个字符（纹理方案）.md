```cpp
#include <gl/glut.h>
#include <iostream>

#include <ft2build.h>
#include FT_FREETYPE_H

using namespace std;

#define PIXEL_SIZE 20 //字体的像素大小
wchar_t ch = L'你';
FT_Library ftLibrary;
FT_Face ftFace;
GLuint  texture;

void print(GLubyte *rgba);

void ConvertMONOToRGBA(FT_Bitmap *source, GLubyte *rgba)
{
	GLubyte *s = source->buffer;
	GLubyte *t = rgba;

	for(GLuint y = source->rows; y > 0; y--)
	{
		GLubyte *ss = s;
		GLubyte *tt = t;

		for(GLuint x = source->width >> 3; x > 0; x--)
		{
			GLuint val = *ss;

			for(GLuint i = 8; i > 0; i--)
			{
				tt[0] = tt[1] = tt[2] = tt[3] = (val & (1 << (i - 1))) ? 0xFF : 0x00;
				tt += 4;
			}

			ss += 1;
		}

		GLuint rem = source->width & 7;

		if(rem > 0)
		{
			GLuint val = *ss;

			for(GLuint x = rem; x > 0; x--)
			{
				tt[0] = tt[1] = tt[2] = tt[3] = (val & 0x80) ? 0xFF : 0x00;
				tt += 4;
				val <<= 1;
			}
		}

		s += source->pitch;
		t += source->width * 4;    //pitch
	}
}

//FreeType的灰色模式：1个像素用1个字节保存。
void ConvertGRAYToRGBA(FT_Bitmap *source, GLubyte *rgba) //灰度图转RGBA
{
	for(GLuint y = 0; y < source->rows; y++)
	{
		for(GLuint x = 0; x < source->width; x++)
		{
			GLubyte *s = &source->buffer[(y * source->pitch) + x];
			GLubyte *t = &rgba[((y * source->pitch) + x) * 4];

			t[0] = t[1] = t[2] = 0xFF; //RGB为255
			t[3] = *s; //A通道存灰度图
		}
	}
}

GLubyte* get_rgba(FT_Bitmap& bitmap)
{
	int rgbaSize = bitmap.rows * bitmap.rows * 4;
	GLubyte *rgba = new GLubyte[rgbaSize];

	switch(bitmap.pixel_mode)     //像素模式
	{
	case FT_PIXEL_MODE_MONO: //黑白
	{
		// 将黑白转成RGBA
		ConvertMONOToRGBA(&bitmap, rgba);
		break;
	}

	case FT_PIXEL_MODE_GRAY: //灰度
	{
		ConvertGRAYToRGBA(&bitmap, rgba); //灰度转RGBA
		break;
	}

	default:
	{
		memset(rgba, 0xFF, rgbaSize);
		break;
	}

	}

	return rgba;
}

GLubyte* get_rgba_2(FT_Bitmap& bitmap) //自己写的，对位图的理解有不足
{
	int rgbaSize = bitmap.rows * bitmap.rows * 4;
	GLubyte *rgba = new GLubyte[rgbaSize];

	for(GLuint y = 0; y < bitmap.rows; y++)
	{
		for(GLuint x = 0; x < bitmap.width; x++)
		{
			GLubyte *s = &bitmap.buffer[y * bitmap.pitch + x / 8];
			GLubyte *t = &rgba[((y * bitmap.width) + x) * 4];

			if((*s & (0xC0 >> (x % 8))) == 0)  //非字符像素
			{
				t[0] = t[1] = t[2] = t[3] = 0x00;
			}
			else //字符像素
			{
				t[0] = t[1] = t[2] = t[3] = 0xff;
			}
		}
	}

	return rgba;
}

void init()
{
	#pragma region FreeType的设置
	// - 初始化FreeType对象
	FT_Init_FreeType(&ftLibrary);

	FT_New_Face(ftLibrary,
				"C:\\WINDOWS\\Fonts\\simhei.ttf", //黑体中文字库
				0,
				&ftFace);

	// - 设置
	FT_Set_Pixel_Sizes(ftFace, PIXEL_SIZE, 0); //像素大小

	// - 得到字符码的字形索引
	FT_UInt uiGlyphIndex = FT_Get_Char_Index(ftFace, ch);

	if(uiGlyphIndex <= 0)
	{
		printf("未找到该字符的索引");
	}

	FT_Load_Glyph(ftFace, uiGlyphIndex, FT_LOAD_DEFAULT);
	#pragma endregion


	#pragma region 生成位图

	// - 生成位图：FreeType所得是8位的灰度图
	if(ftFace->glyph->format != FT_GLYPH_FORMAT_BITMAP)
	{
		FT_Render_Glyph(ftFace->glyph, FT_RENDER_MODE_MONO);
	}

	// - 查看位图信息
	printf("行：%d;列：%d\n", ftFace->glyph->bitmap.rows, ftFace->glyph->bitmap.width);
	printf("\n------ 位图信息 ------\n");

	for(int iRow = 0; iRow < ftFace->glyph->bitmap.rows; iRow++)
	{
		for(int iCol = 0; iCol < ftFace->glyph->bitmap.width; iCol++)
		{
			printf("%d\t", ftFace->glyph->bitmap.buffer[iRow * ftFace->glyph->bitmap.pitch + iCol / 8]);
		}

		printf("\n");
	}

	#pragma endregion

	#pragma region 打印成字体
	printf("\n------ 查看字体信息 ------\n");

	for(int iRow = 0; iRow < ftFace->glyph->bitmap.rows; iRow++)
	{
		for(int iCol = 0; iCol < ftFace->glyph->bitmap.width; iCol++)
		{
			auto index = iRow * ftFace->glyph->bitmap.pitch + iCol / 8;
			auto tmp = 0xC0 >> (iCol % 8);
			auto value = index & tmp;

			if((ftFace->glyph->bitmap.buffer[iRow * ftFace->glyph->bitmap.pitch + iCol / 8] & (0xC0 >> (iCol % 8))) == 0)
			{
				printf(" ");
			}
			else
			{
				printf("1");
			}
		}

		printf("\n");
	}

	#pragma endregion

	#pragma region 生成纹理
	// 生成纹理
	glEnable(GL_TEXTURE_2D);
	glGenTextures(1, &texture);
	glBindTexture(GL_TEXTURE_2D, texture);

	GLubyte* rgba = get_rgba(ftFace->glyph->bitmap);
	//GLubyte* rgba = get_rgba_2(ftFace->glyph->bitmap);

	print(rgba);
	glTexImage2D(GL_TEXTURE_2D,
				 0,
				 GL_RGBA,
				 ftFace->glyph->bitmap.width, ftFace->glyph->bitmap.rows,
				 0,
				 GL_RGBA,
				 GL_UNSIGNED_BYTE,
				 rgba);
	delete rgba;

	glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
	glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);

	glBindTexture(GL_TEXTURE_2D, 0); //解绑
	#pragma endregion

	// 清空资源
	FT_Done_Face(ftFace);
	FT_Done_FreeType(ftLibrary);
}

void myDisplay(void)
{
	glClear(GL_COLOR_BUFFER_BIT);
	glClearColor(0, 0, 0, 0);

	// 画纹理
	glBindTexture(GL_TEXTURE_2D, texture);
	glBegin(GL_QUADS);
	// 要注意字体纹理和OpenGL的绘制关系
	glTexCoord2d(0, 0);         //纹理 左下角
	glVertex2f(-0.8f, 0.8f);    //OpenGL 左上角

	glTexCoord2d(0, 1);         //左上角
	glVertex2f(-0.8f, -0.8f);   //左下角

	glTexCoord2f(1.0f, 1.0f);   //右上角
	glVertex2f(0.8f, -0.8f);    //右下角

	glTexCoord2d(1, 0);         //右下角
	glVertex2f(0.8f, 0.8f);     //右上角

	glEnd();
	glFlush();
}

int main(int argc, char *argv[])
{
	glutInit(&argc, argv);                          //初始化GLUT
	glutInitDisplayMode(GLUT_RGB | GLUT_SINGLE);    //初始化显示模式
	glutInitWindowPosition(100, 100);               //窗体位置
	glutInitWindowSize(500, 500);                   //窗体大小
	glutCreateWindow("OpenGL One Char");            //传入窗口名称，并开窗

	init();
	glutDisplayFunc(&myDisplay);                    //绘制的回调函数

	glutMainLoop();                                 //循环绘制
	return 0;
}

void print(GLubyte *rgba) //打印RGBA
{
	printf("\n----- R -----\n");

	for(GLuint y = 0; y < ftFace->glyph->bitmap.rows; y++)
	{
		for(GLuint x = 0; x < ftFace->glyph->bitmap.width; x++)
		{
			GLubyte *s = &ftFace->glyph->bitmap.buffer[(y * ftFace->glyph->bitmap.pitch) + x];
			GLubyte *t = &rgba[((y * ftFace->glyph->bitmap.pitch) + x) * 4];

			printf("%d\t", t[0]);
		}

		printf("\n");
	}

	printf("\n----- G -----\n");

	for(GLuint y = 0; y < ftFace->glyph->bitmap.rows; y++)
	{
		for(GLuint x = 0; x < ftFace->glyph->bitmap.width; x++)
		{
			GLubyte *s = &ftFace->glyph->bitmap.buffer[(y * ftFace->glyph->bitmap.pitch) + x];
			GLubyte *t = &rgba[((y * ftFace->glyph->bitmap.pitch) + x) * 4];

			printf("%d\t", t[1]);
		}

		printf("\n");
	}

	printf("\n----- B -----\n");

	for(GLuint y = 0; y < ftFace->glyph->bitmap.rows; y++)
	{
		for(GLuint x = 0; x < ftFace->glyph->bitmap.width; x++)
		{
			GLubyte *s = &ftFace->glyph->bitmap.buffer[(y * ftFace->glyph->bitmap.pitch) + x];
			GLubyte *t = &rgba[((y * ftFace->glyph->bitmap.pitch) + x) * 4];

			printf("%d\t", t[2]);
		}

		printf("\n");
	}

	printf("\n----- A -----\n");

	for(GLuint y = 0; y < ftFace->glyph->bitmap.rows; y++)
	{
		for(GLuint x = 0; x < ftFace->glyph->bitmap.width; x++)
		{
			GLubyte *s = &ftFace->glyph->bitmap.buffer[(y * ftFace->glyph->bitmap.pitch) + x];
			GLubyte *t = &rgba[((y * ftFace->glyph->bitmap.pitch) + x) * 4];

			printf("%d\t", t[3]);
		}

		printf("\n");
	}
}
```