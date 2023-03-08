```cpp
#include<stdio.h>
#include <ft2build.h>
#include FT_FREETYPE_H

#define CHARSIZE 32 // 字符位图的大小设为32 * 32

int GetCharBitmap(int iCharSize, unsigned int uiCharCode);

int main()
{
	//wchar_t ch = 'b'; //英文字符
	wchar_t ch = L'总'; //中文字符
	//wchar_t ch = '你';
	/* 中文字符必须要加L，不然得不到字符索引FT_Get_Char_Index()返回0
	 * FreeType默认为unicode的字符表
	 * L将ANSI字符串转换成unicode的字符串
	 *
	 */

	GetCharBitmap(CHARSIZE, ch);

	return 0;
}

int GetCharBitmap(int iCharSize, unsigned int uiCharCode)
{
	// - 初始化FreeType对象
	FT_Library ftLibrary;
	FT_Error ftError = FT_Init_FreeType(&ftLibrary);

	if(ftError)
	{
		printf("Init freetype library fail!/n");
		return -1;
	}

	// - 从字体文件中装载一个Face
	FT_Face ftFace;
	ftError = FT_New_Face(ftLibrary,
						  "C:\\WINDOWS\\Fonts\\simhei.ttf", //黑体中文字库
						  0,
						  &ftFace);

	if(ftError == FT_Err_Unknown_File_Format)
	{
		// 表示可以打开和读此文件，但不支持此字体格式
		printf("Error! Could not support this format!/n");
		return -1;

	}
	else if(ftError) // 其他错误
	{
		printf("Error! Could not open file ukai.ttc!/n");
		return -1;
	}

	// - 设置
	ftError = FT_Set_Pixel_Sizes(ftFace, iCharSize, 0);

	if(ftError)
	{
		printf("Set pixel sizes to %d*%d error!/n", iCharSize, iCharSize);
		return -1;
	}

	// - 得到字符码的字形索引
	FT_UInt uiGlyphIndex = FT_Get_Char_Index(ftFace, uiCharCode);

	if(uiGlyphIndex<=0)
	{
		printf("未找到该字符的索引");
		return -1;
	}

	FT_Load_Glyph(ftFace, uiGlyphIndex, FT_LOAD_DEFAULT);

	// - 生成位图
	if(ftFace->glyph->format != FT_GLYPH_FORMAT_BITMAP)
	{
		FT_Render_Glyph(ftFace->glyph, FT_RENDER_MODE_MONO);
	}

	int iRow = 0, iCol = 0;

	for(iRow = 0; iRow < ftFace->glyph->bitmap.rows; iRow++)
	{
		for(iCol = 0; iCol < ftFace->glyph->bitmap.width; iCol++)
		{
			if((ftFace->glyph->bitmap.buffer[iRow * ftFace->glyph->bitmap.pitch + iCol / 8] & (0xC0 >> (iCol % 8))) == 0)
			{
				printf("1");
			}
			else
			{
				printf("0");
			}
		}

		printf("\n");
	}

	return 0;

}
```