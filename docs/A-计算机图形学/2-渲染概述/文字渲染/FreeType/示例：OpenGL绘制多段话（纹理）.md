```cpp
/*!
 * \file:   Exp1.cpp
 * \date:   2020/05/29 15:04
 * \author: wangqi
 * Contact: wangq@southzn.com
 *
 * \brief:
 *
 * TODO: long description
 *
 * \note:   实现了FT库生成字符位图，并上传到GL纹理。
            实现了字符位图缓存功能，多个字符图像保存在同一个纹理中。
            实现了简单的字体管理框架。
            实现了简单的加粗和倾斜效果。
            实现了反锯齿开关，并且兼容加粗倾斜效果。
 */

// OpenGL library
#include <gl/glut.h>

// Std misc
#include <map>
#include <vector>

// FreeType library
#include <ft2build.h>
#include FT_FREETYPE_H
#include FT_BITMAP_H
#include FT_OUTLINE_H


#ifdef CreateFont
#undef CreateFont
#endif

typedef unsigned char byte;

class CFontManager //字体管理器：管理了一个字体库
{
public:
	CFontManager();
	~CFontManager();
	// 初始化
	bool initialize(void);
	// 释放
	void release(void);
	/*!
	 * \brief:   createFont 创建字体
	 *
	 * \param:   filename   字体文件的路径
	 * \param:   face
	 * \param:   tall       高度
	 * \param:   bold       加粗
	 * \param:   italic     是否斜体
	 * \param:   antialias  是否抗锯齿
	 * \returns: int
	 * \author:  wangqi
	 * \date:    2020/06/02 14:16
	 *
	 * TODO:
	 *
	 */
	int createFont(const char *filename, int face, int tall, bool bold, bool italic, bool antialias);
	// 得到字符信息
	bool getCharInfo(int font_index, int code, int *wide, int *tall, int *horiBearingX, int *horiBearingY, int *horiAdvance, GLuint *texture, float coords[]);
	// 得到字体的高度
	int getFontTall(int font_index);

private:
	struct glyphMetrics //字形度量
	{
		int    width;   //宽度
		int    height;  //长度
		int    horiBearingX;
		int    horiBearingY;
		int    horiAdvance;
		//int    vertBearingX;
		//int    vertBearingY;
		//int    vertAdvance;
	};

	class CFont //字体
	{
	public:
		CFont();
		~CFont();
		/*!
		 * \brief:   create     创建字体
		 *
		 * \param:   library    FT库。需要提前FT_Init_FreeType
		 * \param:   filename   字体文件的路径
		 * \param:   face_index Face下标
		 * \param:   tall       高度
		 * \param:   bold       是否加粗
		 * \param:   italic     是否斜体
		 * \param:   antialias  是否抗锯齿
		 * \returns: bool
		 * \author:  wangqi
		 * \date:    2020/06/02 14:13
		 *
		 * TODO:
		 *
		 */
		bool create(FT_Library library, const char *filename, FT_Long face_index,
					int tall, bool bold, bool italic, bool antialias);
		// 释放
		void release(void);
		// 得到字符串信息
		bool getCharInfo(int code, glyphMetrics *metrics, GLuint *texture, float coords[]);
		// 得到字符高度
		int getFontTall(void);

	private:
		// 加载字符
		bool loadChar(int code, glyphMetrics *metrics);

		class CChar //渲染之后的字符
		{
		public:
			// 设置信息
			void setInfo(glyphMetrics *metrics);
			// 得到信息
			void getInfo(glyphMetrics *metrics, GLuint *texture, float coords[]);

		public:
			int m_code;             //编码
			GLuint  m_texture;      //纹理
			float   m_coords[4];    //坐标：left top right bottom

		private:
			glyphMetrics    m_metrics;
		};

		class CPage //页面:纹理=1:1。一个纹理中管理着多个字符
		{
		public:
			CPage();
			~CPage();

			/*!
			 * \brief:   append     加当前字体加入这个纹理
			 *
			 * \param:   wide       宽度
			 * \param:   tall       高度
			 * \param:   rgba       字体的内容
			 * \param:   coords     字体在当前纹理中的坐标
			 * \returns: bool
			 * \author:  wangqi
			 * \date:    2020/06/02 14:36
			 *
			 * TODO:
			 *
			 */
			bool append(int wide, int tall, byte *rgba, float coords[]);
			GLuint getTexture(void); //得到纹理

		private:
			GLuint    m_texture; //纹理
			int        m_wide;  //宽度
			int        m_tall;  //高度
			int        m_posx;
			int        m_posy;
			int        m_maxCharTall;
		};

		typedef std::map<int, CChar *> TCharMap;
		//键值对：找到字符的位置
		//key: 字符
		//value: 字符的信息

		FT_Library                m_library;
		FT_Face                    m_face;
		bool                    m_antialias;
		bool                    m_bold;
		int                        m_tall;
		int                        m_rgbaSize;
		GLubyte                    *m_rgba;
		TCharMap                m_chars;
		std::vector<CPage *>    m_pages;
	};

	FT_Library                m_library;
	std::vector<CFont *>      m_fonts; //一个字体库有多个字体
};

//------------------------------------------------------------
// CFont
//------------------------------------------------------------
CFontManager::CFont::CFont()
{
	m_face = NULL;
	m_rgba = NULL;
	m_antialias = false;
	m_bold = false;
	m_tall = 0;
}

CFontManager::CFont::~CFont()
{
	release();
}

bool CFontManager::CFont::create(FT_Library library, const char *filename, FT_Long face_index,
								 int tall, bool bold, bool italic, bool antialias)
{
	FT_Error err;

	if(tall > 256)
	{
		// Bigger than a page size
		return false;
	}

	// 得到字符信息
	if((err = FT_New_Face(library, filename, face_index, &m_face)) != FT_Err_Ok)
	{
		printf("FT_New_Face() Error %d\n", err);
		return false;
	}

	// 设置像素
	if((err = FT_Set_Pixel_Sizes(m_face, 0, tall)) != FT_Err_Ok)
	{
		printf("FT_Set_Pixel_Sizes() Error %d\n", err);
		return false;
	}

	//m_rgbaSize = (tall * 2) * tall * 4; //源代码是*2，但是觉得博主写错了
	m_rgbaSize = (tall) * tall * 4; //高*宽*4。但位图的高宽相同，故可以高*高*4；4代表RGBA四个通道

	m_rgba = new GLubyte[m_rgbaSize];

	if(m_rgba == NULL)
	{
		return false;
	}

	m_library = library;
	m_antialias = antialias;
	m_bold = bold;
	m_tall = tall;

	if(italic) //是否抗锯齿
	{
		FT_Matrix m;
		m.xx = 0x10000L;
		m.xy = 0.5f * 0x10000L;
		m.yx = 0;
		m.yy = 0x10000L;
		FT_Set_Transform(m_face, &m, NULL);
	}

	return true;
}

void CFontManager::CFont::release(void)
{
	FT_Error err;

	if(m_face)
	{
		if((err = FT_Done_Face(m_face)) != FT_Err_Ok)
		{
			printf("FT_Done_Face() Error %d\n", err);
		}

		m_face = NULL;
	}

	if(m_rgba)
	{
		delete[] m_rgba;
		m_rgba = NULL;
	}

	for(TCharMap::iterator it = m_chars.begin(); it != m_chars.end(); it++)
	{
		delete it->second;
		it->second = NULL;
	}

	m_chars.clear();

	for(size_t i = 0; i < m_pages.size(); i++)
	{
		delete m_pages[i];
		m_pages[i] = NULL;
	}

	m_pages.clear();
}

bool CFontManager::CFont::getCharInfo(int code, glyphMetrics *metrics, GLuint *texture, float coords[])
{
	// 记忆化搜索
	TCharMap::iterator it = m_chars.find(code); //根据code获取CChar

	if(it != m_chars.end()) //找到
	{
		// 得到绘制信息
		it->second->getInfo(metrics, texture, coords); //字符、纹理、 坐标
		return true;
	}

	// 没有找到
	glyphMetrics gm;

	if(loadChar(code, &gm) == false) //加载字符
	{
		return false;
	}

	CChar *ch = new CChar();

	ch->m_code = code;
	ch->setInfo(&gm);

	// 追加到一张纸中
	for(size_t i = 0; i < m_pages.size(); i++)
	{
		CPage *page = m_pages[i];

		if(page->append(gm.width, gm.height, m_rgba, ch->m_coords))
		{
			// 这张纸还放的下
			ch->m_texture = page->getTexture();
			ch->getInfo(metrics, texture, coords);
			m_chars.insert(TCharMap::value_type(code, ch));
			return true;
		}
	}

	// 之前的纸都放不下了，新建一张纸
	CPage *page = new CPage();

	if(page->append(gm.width, gm.height, m_rgba, ch->m_coords))
	{
		ch->m_texture = page->getTexture();
		ch->getInfo(metrics, texture, coords);
		m_chars.insert(TCharMap::value_type(code, ch));
		m_pages.push_back(page);
		return true;
	}

	delete ch;
	delete page;

	return false;
}

int CFontManager::CFont::getFontTall(void)
{
	return m_tall;
}

// bitmap.width  位图宽度
// bitmap.rows   位图行数（高度）
// bitmap.pitch  位图一行占用的字节数

//MONO模式每1个像素仅用1bit保存，只有黑和白。
//1个byte可以保存8个像素，1个int可以保存8*4个像素。
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

//GRAY模式1个像素用1个字节保存。
void ConvertGRAYToRGBA(FT_Bitmap *source, GLubyte *rgba)
{
	for(GLuint y = 0; y < source->rows; y++)
	{
		for(GLuint x = 0; x < source->width; x++)
		{
			GLubyte *s = &source->buffer[(y * source->pitch) + x];
			GLubyte *t = &rgba[((y * source->pitch) + x) * 4];

			t[0] = t[1] = t[2] = 0xFF;
			t[3] = *s;
		}
	}
}

bool CFontManager::CFont::loadChar(int code, glyphMetrics *metrics)
{
	FT_Error err;

	FT_UInt glyph_index = FT_Get_Char_Index(m_face, (FT_ULong)code); //得到下标

	if(glyph_index > 0)
	{
		//将一个字形图像装到字形槽中
		if((err = FT_Load_Glyph(m_face, glyph_index, FT_LOAD_DEFAULT)) == FT_Err_Ok)
		{
			FT_GlyphSlot glyph = m_face->glyph; //获得图形信息
			// 模式：抗锯齿为256级灰度;FT_RENDER_MODE_MONO黑白
			FT_Render_Mode render_mode = m_antialias ? FT_RENDER_MODE_NORMAL : FT_RENDER_MODE_MONO;

			if(m_antialias && m_bold) //加粗
			{
				if((err = FT_Outline_EmboldenXY(&glyph->outline, 60, 60)) != FT_Err_Ok)
				{
					printf("FT_Outline_EmboldenXY() Error %d\n", err);
				}
			}

			// 得到图形信息
			if((err = FT_Render_Glyph(glyph, render_mode)) == FT_Err_Ok)
			{
				FT_Bitmap *bitmap = &glyph->bitmap; //得到位图

				switch(bitmap->pixel_mode) //像素模式
				{
				case FT_PIXEL_MODE_MONO: //黑白
				{
					if(!m_antialias && m_bold)
					{
						// 加粗
						if((err = FT_Bitmap_Embolden(m_library, bitmap, 60, 0)) != FT_Err_Ok)
						{
							printf("FT_Bitmap_Embolden() Error %d\n", err);
						}
					}

					// 将黑白转成RGBA
					ConvertMONOToRGBA(bitmap, m_rgba);
					break;
				}

				case FT_PIXEL_MODE_GRAY: //灰度
				{
					ConvertGRAYToRGBA(bitmap, m_rgba); //灰度转RGBA
					break;
				}

				default:
				{
					memset(m_rgba, 0xFF, m_rgbaSize);
					break;
				}
				}

				metrics->width = bitmap->width;
				metrics->height = bitmap->rows;
				metrics->horiBearingX = glyph->bitmap_left;
				metrics->horiBearingY = glyph->bitmap_top;
				metrics->horiAdvance = glyph->advance.x >> 6;

				return true;
			}
			else
			{
				printf("FT_Render_Glyph() Error %d\n", err);
			}
		}
		else
		{
			printf("FT_Load_Glyph() Error %d\n", err);
		}
	}

	memset(metrics, 0, sizeof(glyphMetrics));

	return false;
}

//------------------------------------------------------------
// CChar
//------------------------------------------------------------
void CFontManager::CFont::CChar::setInfo(glyphMetrics *metrics)
{
	memcpy(&m_metrics, metrics, sizeof(glyphMetrics));
}

void CFontManager::CFont::CChar::getInfo(glyphMetrics *metrics, GLuint *texture, float coords[])
{
	memcpy(metrics, &m_metrics, sizeof(glyphMetrics));

	*texture = m_texture;
	memcpy(coords, m_coords, sizeof(float) * 4);
}

//------------------------------------------------------------
// CPage 开辟一页的纹理，来存放多个字符
//------------------------------------------------------------
CFontManager::CFont::CPage::CPage() //一页对应一个纹理
{
	m_wide = m_tall = 256; //高
	m_posx = m_posy = 0;

	// In a line, for a max height character
	m_maxCharTall = 0;

	glGenTextures(1, &m_texture);    // Using your API here
	glBindTexture(GL_TEXTURE_2D, m_texture);
	glTexImage2D(GL_TEXTURE_2D, 0, GL_RGBA, m_wide, m_tall, 0, GL_RGBA, GL_UNSIGNED_BYTE, NULL);
	glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
	glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
}

CFontManager::CFont::CPage::~CPage()
{
	// free the texture
}

bool CFontManager::CFont::CPage::append(int wide, int tall, byte *rgba, float coords[])
{
	if(m_posy + tall > m_tall)
	{
		// not enough line space in this page
		return false;
	}

	// If this line is full ...
	if(m_posx + wide > m_wide)
	{
		int newLineY = m_posy + m_maxCharTall;

		if(newLineY + tall > m_tall)
		{
			// No more space for new line in this page, should allocate a new one
			return false;
		}

		// Begin in new line
		m_posx = 0;
		m_posy = newLineY;
		// Reset
		m_maxCharTall = 0;
	}

	glBindTexture(GL_TEXTURE_2D, m_texture);
	glTexSubImage2D(GL_TEXTURE_2D, 0, m_posx, m_posy, wide, tall, GL_RGBA, GL_UNSIGNED_BYTE, rgba);
	// 该字符的位置
	coords[0] = m_posx / (float)m_wide;                // left
	coords[1] = m_posy / (float)m_tall;                // top
	coords[2] = (m_posx + wide) / (float)m_wide;    // right
	coords[3] = (m_posy + tall) / (float)m_tall;    // bottom

	m_posx += wide;

	if(tall > m_maxCharTall)
	{
		m_maxCharTall = tall;
	}

	return true;
}

GLuint CFontManager::CFont::CPage::getTexture(void)
{
	return m_texture;
}

//------------------------------------------------------------
// CFontManager
//------------------------------------------------------------
CFontManager::CFontManager()
{
	m_library = NULL;
}

CFontManager::~CFontManager()
{
	release();
}

bool CFontManager::initialize(void)
{
	FT_Error err;

	if((err = FT_Init_FreeType(&m_library)) != FT_Err_Ok) //初始化一个库
	{
		printf("FT_Init_FreeType() Error %d\n", err);
		return false;
	}

	return true;
}

void CFontManager::release(void)
{
	FT_Error err;

	for(size_t i = 0; i < m_fonts.size(); i++)
	{
		delete m_fonts[i];
		m_fonts[i] = NULL;
	}

	m_fonts.clear();

	if((err = FT_Done_FreeType(m_library)) != FT_Err_Ok)
	{
		printf("FT_Done_FreeType() Error %d\n");
	}
}

int CFontManager::createFont(const char *filename, int face, int tall, bool bold,
							 bool italic, bool antialias)
{
	// 新建字体
	CFont *font = new CFont();

	if(font->create(m_library, filename, face, tall, bold, italic, antialias) != true)
	{
		delete font; //创建失败
		return 0;
	}

	// 放入字体数组，进行管理
	m_fonts.push_back(font);

	return m_fonts.size();
}

#define CONVERT_FONT_INDEX(x) (((x) < 1 || (x) > (int)m_fonts.size()) ? -1 : (x) - 1)

bool CFontManager::getCharInfo(int font_index, int code, int *wide, int *tall, int *horiBearingX, int *horiBearingY, int *horiAdvance, GLuint *texture, float coords[])
{
	int i = CONVERT_FONT_INDEX(font_index); //转换字符的下标

	if(i == -1)
	{
		return false;
	}

	CFont *font = m_fonts[i]; //获得字体

	glyphMetrics metrics; //绘制信息

	if(font->getCharInfo(code, &metrics, texture, coords) == false) //获得绘制信息
	{
		return false;
	}

	// 解析绘制信息
	*wide = metrics.width;
	*tall = metrics.height;
	*horiBearingX = metrics.horiBearingX;
	*horiBearingY = metrics.horiBearingY;
	*horiAdvance = metrics.horiAdvance;

	return true;
}

int CFontManager::getFontTall(int font_index)
{
	int i = CONVERT_FONT_INDEX(font_index);

	if(i == -1)
	{
		return false;
	}

	CFont *font = m_fonts[i];

	return font->getFontTall();
}

CFontManager g_FontManager;


int char_font;


void init(void)
{
	// 清除颜色
	glClearColor(0.0, 0.0, 0.0, 0.0);
	// 混合模式
	glEnable(GL_BLEND);
	glBlendFunc(GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA);
	// 初始化一个库
	g_FontManager.initialize();

	// 创建字体，返回字体集的个数
	char_font = g_FontManager.createFont("C:\\WINDOWS\\Fonts\\simhei.ttf",
										 0,
										 32, //字体高度
										 false,
										 false,
										 true);

	if(char_font == 0)//创建失败
	{
		printf("createFont failed\n");
	}
}

wchar_t ciphertext[] =
{
	L"你好吗？\n"
	L"温馨提示：\n"
	L"不要忘记明天上午 7:30-12:00 在公司的体检哦~\n"
	L"体检前一天不要暴饮暴食，不要喝酒，不要进食太甜太咸的食物\n"
	L"以免影响体检结果\n"
	L"晚上10点后一般要求禁食。\n"
	L"体检当天早晨千万不要吃早餐！\n"
	L"不要吃早餐！\n"
	L"不要吃早餐！\n"
	L"绘制位置。忘记了，可费解为无法\n"
};

//#define DRAW_PAGE

/*!
 * \brief:   draw_string
 *
 * \param:   x              绘制位置
 * \param:   y
 * \param:   font           字体
 * \param:   string         字符串
 * \returns: void
 * \author:  wangqi
 * \date:    2020/06/02 14:18
 *
 * TODO:
 *
 */
void draw_string(int x, int y, int font_index, wchar_t *string)
{
	if(!font_index) //字体
	{
		return;
	}

	int tall = g_FontManager.getFontTall(font_index); //得到字符高：初始化字体时设置的

	// 绘制的初始位置
	int dx = x;
	int dy = y;

	GLuint sglt = 0; //纹理编号，第几张纸

	while(*string) // 遍历字符串
	{
		if(*string == L'\n') //回车
		{
			string++;
			dx = x;         //绘制的初始位置x
			dy += tall + 2; //行间距
			continue;
		}

		int cw, ct; //宽度、高度
		int bx, by, av;
		GLuint glt;   //该字符所在的纸张
		float crd[4]; //该字符的坐标

		//得到字符信息
		if(!g_FontManager.getCharInfo(font_index, *string, &cw, &ct, &bx, &by, &av, &glt, crd))
		{
			string++;
			continue;
		}

		//大多数情况下多个字符都在同一个纹理中，避免频繁绑定纹理，可以提高效率
		if(glt != sglt) //和原来的不一样，则绑定当前的纹理
		{
			glBindTexture(GL_TEXTURE_2D, glt); //绑定纹理
			sglt = glt;
		}

		int px = dx + bx;
		int py = dy - by;

		// 开始绘制四边形
		glBegin(GL_QUADS);
		// 第一个点
		glTexCoord2f(crd[0], crd[1]);   //指定该字符在纹理中的坐标，阈值为[0, 1]
		glVertex3f(px, py, 0.0f);       //对应画布中的位置
		// 第二个点
		glTexCoord2f(crd[2], crd[1]);
		glVertex3f(px + cw, py, 0.0f);
		// 第三个点
		glTexCoord2f(crd[2], crd[3]);
		glVertex3f(px + cw, py + ct, 0.0f);
		// 第四个点
		glTexCoord2f(crd[0], crd[3]);
		glVertex3f(px, py + ct, 0.0f);
		// 结束绘制
		glEnd();

		dx += av;
		string++;
	}
}

void draw_page_texture(int x, int y, GLuint glt)
{
	if(!glt)
	{
		glDisable(GL_TEXTURE_2D); //关闭
		glColor4f(0.2, 0.2, 0.2, 1.0);
	}
	else
	{
		glEnable(GL_TEXTURE_2D); //开启
		glBindTexture(GL_TEXTURE_2D, glt); //绑定纹理
		glColor4f(1.0, 1.0, 1.0, 1.0);
	}

	int w = 256;
	int t = 256;

	glBegin(GL_QUADS);
	glTexCoord2f(0.0, 0.0);
	glVertex3f(x, y, 0.0f);
	glTexCoord2f(1.0, 0.0);
	glVertex3f(x + w, y, 0.0f);
	glTexCoord2f(1.0, 1.0);
	glVertex3f(x + w, y + t, 0.0f);
	glTexCoord2f(0.0, 1.0);
	glVertex3f(x, y + t, 0.0f);
	glEnd();
}

void display(void)
{
	// 清除缓存
	glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT | GL_STENCIL_BUFFER_BIT);
	// 开启纹理
	glEnable(GL_TEXTURE_2D);
	// 设置画笔颜色，即字体颜色
	glColor4f(0.0, 0.0, 1.0, 1.0);
	// 绘制字符串
	draw_string(10, 30, char_font, ciphertext);

	// 绘制整个纹理
	draw_page_texture(10, 350, 0);    //绘制一个背景
	draw_page_texture(10, 350, 1);    //绘制第一张纸（即第一个纹理）

	draw_page_texture(276, 350, 0);   //绘制一个背景
	draw_page_texture(276, 350, 2);   //绘制第二张纸（即第二个纹理）

	glutSwapBuffers();
	glutPostRedisplay();
}

void reshape(int width, int height)
{
	glViewport(0, 0, width, height);

	glMatrixMode(GL_PROJECTION);
	glLoadIdentity();
	gluOrtho2D(0, width, height, 0);
	glMatrixMode(GL_MODELVIEW);
}

void keyboard(unsigned char key, int x, int y)
{
}

void main(int argc, char **argv)
{
	glutInitWindowPosition(200, 200);
	glutInitWindowSize(1200, 680);
	glutInit(&argc, argv);
	glutInitDisplayMode(GLUT_RGBA | GLUT_DOUBLE | GLUT_DEPTH | GLUT_STENCIL);
	glutCreateWindow("FreeType OpenGL");

	init();
	glutDisplayFunc(display);

	glutReshapeFunc(reshape);
	glutKeyboardFunc(keyboard);
	glutMainLoop();
}
```