FreeType绘制的基本流程例子：矢量文字的实现过程：

1. 给定一个文字，无论是神马编码方式（ASCII、GBK、unicode、BIG5），都可以确定他的编码值
2. 根据编码值从字体文件中找到“glyph”
3. 设置字体大小
4. 用某些函数把glyph里的点缩放为设置的字体大小
5. 转换为位图点阵
6. 在LCD上显示出来

## 初始化FreeType2对象（FT_Library）
```cpp
FT_EXPORT(FT_Error)  FT_Init_FreeType( FT_Library *alibrary );
```

## 装载字体（Face）
一个Face对象描述了一个特定的字样和风格
```cpp
// 方法一：从字体文件装载一个字体Face
FT_EXPORT(FT_Error) FT_New_Face(FT_Library   library,		// 库的句柄
                            	const char* filepathname, 	// 字体文件路径名
                            	FT_Long      face_index,    // 字体face的索引,该索引指示你想装载的face如果这个值太大，函数将会返回一个错误Index为0总是正确的。
                            	FT_Face     *aface );       // 一个指向新建的face对象的指针
// 方法二：从内存装载Face
FT_EXPORT(FT_Error) FT_New_Memory_Face(	FT_Library      library,    // 库的句柄
                                       	const FT_Byte* file_base, 	// 指向字体数据的第一个字节
                                       	FT_Long         file_size, 	// 缓存的大小（以字节表示）
                                       	FT_Long         face_index, // face索引
                                       	FT_Face*		aface );    // face对象的指针
```

## 设置相关参数
### 设置字符像素大小（FT_Set_Pixel_Sizes、FT_Set_Char_Size）
若像素宽度、高度有一个为0，则表示按照字形来反算出另一个。
```cpp
FT_EXPORT(FT_Error)	FT_Set_Pixel_Sizes( FT_Face face,            // face对象句柄
                                        FT_UInt pixel_width,     // 像素宽度
                                        FT_UInt pixel_height );  // 像素高度

FT_EXPORT(FT_Error) FT_Set_Char_Size(FT_Face face,
                         			FT_F26Dot6 char_width,  //以1/64点为单位的字符宽度
                                    FT_F26Dot6 char_height, //以1/64点为单位的字符高度
                             		FT_UInthorz_resolution, //设备水平分辨率
                                    FT_UIntvert_resolution )： //设备垂直分辨率
```

### 设置字符宽高及分辨率（FT_Set_Char_Size）
```cpp
FT_Set_Char_Size(face, 0, 16*64, 300, 300);
```

## 得到字符码的字形索引（FT_Get_Char_Index）
【装载一个字形图像】把一个字符码转换为一个字形索引，并获得字体的点阵。这里的字符码为Unicode

【说明】很多TrueType字体包含两个字符表。但是当新建一个face对象时，它默认选择Unicode字符表

1. 一个用来转换Unicode字符码到字形索引
2. 另一个用来转换Apple Roman编码到字形索引

```cpp
// - 得到uiCharCode字符的字形索引
FT_UInt uiGlyphIndex = FT_Get_Char_Index(ftFace, uiCharCode);
	// unsigned int uiCharCode = 0x0061;  //字母a的ucs2编码
// - 设置字符表
FT_Select_Charmap(face,FT_ENCODING_UNICODE);

```

## 获得该字符的图像（字行槽，glyph slot）
【补充知识】在不同字体中，字形图像存储为不同的格式

1. 对于固定尺寸字体格式，每一图像都是一个位图
2. 对于可伸缩字体格式，使用名为轮廓（outlines）的矢量形状来描述每一个字形

【背景】在获得字形索引之后，便可以装载该字符的图像

【字形槽，glyph slot】字形图像存储在一个特别的对象（字形槽，glyph slot）中

1. 正如其名，一个字形槽只是一个简单的容器，它一次只能容纳一个字形图像
2. 每一个face对象都有一个字形槽对象，可以通过`face->glyph`来访问
```cpp
// 将一个字形图像装到字形槽中
 FT_EXPORT(FT_Error) FT_Load_Glyph(FT_Face face,         //face对象的句柄
								   FT_UInt glyph_index,  //字形索引
								   FT_Int32 load_flags ); //装载标志,其默认值是FT_LOAD_DEFAULT即0
// 获得当前字形槽中存储字形图像的格式
face->glyph->format //返回值：FT_GLYPH_FORMAT_BITMAP
```
【装载标志load_flags】

1. FT_LOAD_DEFAULT
2. FT_LOAD_RENDER：字形图像必须立即转换为一个抗锯齿位图。这是一个捷径，可以取消明显的调用FT_Render_Glyph，但功能是相同的。

【FT_Loac_Char】使用函数FT_Loac_Char代替FT_Load_Glyph。如你大概想到的，它相当于先调用GT_Get_Char_Index然后调用FT_Load_Glyph

### 位图（FT_Bitmap_）
位图：即字体的点阵。缺点比较明显，缩放存在锯齿，渲染旋转等操作相对复杂，且效果不理想，先大多用在嵌入式行业（基本抛弃），常见格式有bdf，pcf，fnt，hbf，hzf等。
```cpp
// 生成位图。调用后，数据存放在face->glyph->bitmap中
FT_EXPORT(FT_Error) FT_Render_Glyph(FT_GlyphSlot slot,            // 字形槽
									FT_Render_Mode render_mode ); // 渲染模式
```

【渲染模式】FT_Render_Mode

1. FT_RENDER_MODE_NORMAL：高质量的抗锯齿（256级灰度）位图
2. FT_RENDER_MODE_MONO：黑白位图
3. FT_RENDER_MODE_LCD：

【FT_Bitmap_的原形】

1. bitmap.width  位图宽度
2. bitmap.rows   位图行数（高度）
3. bitmap.pitch  它的绝对值是位图一行所占用的字节数，根据位图向量方向，可以是正值或是负值
```cpp
typedef struct FT_Bitmap_{
	int             rows;           // 位图的行数
	int             width;          // 位图的宽度,也表示每行有多少个像素
	int             pitch;          // 偏移值,当往上时为负数,往下时为正数,它的绝对值为位图的一行所占的字节数.不够一个字节当一个字节算.
	unsigned char*  buffer;         // 指向位图缓冲区的指针
	short           num_grays;      // 该值只应用于FT_PIXEL_MODE_GRAY模式
	char            pixel_mode;     // 像素模式
	char            palette_mode;   // 色块像素模式
	void*           palette;        // 调色板
} FT_Bitmap;
```

### 轮廓
轮廓字体：即矢量字体，利用字体轮廓及填充实现字体显示，优势明显，渲染缩放较容易，但效率相对低些（相对于嵌入式）

## 收尾工作
```cpp
FT_Done_Glyph(glyph);
FT_Done_Face(pFTFace);
FT_Done_FreeType(pFTLib);
```

## 显示
### 通过GDI+绘制出位图
```cpp
// 创建内存位图
unsigned  char *pvBits =  new unsigned  char [10000];
HBITMAP hBitmap =CreateDIBSection(NULL, &bmpinfo, DIB_RGB_COLORS, ( void ** )&pvBits, NULL, 0 );
int iLineBytes = (bmpinfo.bmiHeader.biWidth + 7) / 8;
for ( int i = 0; i != bmpinfo.bmiHeader.biHeight; ++i)
{
	memcpy (pvBits + i * iLineBytes, face->glyph->bitmap.buffer + i * iLineBytes, iLineBytes);
}

Bitmap *pBitmap = Bitmap::FromHBITMAP(hBitmap, NULL);
Graphics graphic(pDC->m_hDC);
graphic.DrawImage(pBitmap, Point(20, 150));
```

### 将位图保存成图片

### 在OpenGL中绘制位图

### 在OpenGL中绘制轮廓
