1 什么是TIFF？  
TIFF是Tagged Image File Format的缩写。在现在的标准中，只有TIFF存在， 其他的提法已经舍弃不用了。做为一种标记语言，TIFF与其他文件格式最大的不同在于除了图像数据，它还可以记录很多图像的其他信息。它记录图像数据的方式也比较灵活， 理论上来说， 任何其他的图像格式都能为TIFF所用， 嵌入到TIFF里面。比如JPEG， Lossless JPEG， JPEG2000和任意数据宽度的原始无压缩数据都可以方便的嵌入到TIFF中去。由于它的可扩展性， TIFF在数字影响、遥感、医学等领域中得到了广泛的应用。TIFF文件的后缀是.tif或者.tiff  
  
2 TIFF文件结构   
TIFF文件中的三个关键词是：图像文件头Image File Header(IFH)， 图像文件目录Image File Directory(IFD)和目录项Directory Entry(DE)。每一幅图像是以8字节的IFH开始的， 这个IFH指向了第一个IFD。IFD包含了图像的各种信息， 同时也包含了一个指向实际图像数据的指针。  
IFH的构成：  
Byte 0-1: 字节顺序标志位， 值为II或者MM。II表示小字节在前， 又称为little-endian。MM表示大字节在前，又成为big-endian。  
Byte 2-3: TIFF的标志位，一般都是42  
Byte 4-7: 第一个IFD的偏移量。可以在任意位置， 但必须是在一个字的边界，也就是说必须是2的整数倍。  
IFD的构成(0代表此IFD的起始位置)：  
Byte 0-1: 表示此IFD包含了多少个DE，假设数目为n  
Byte 2-(n*12+1): n个DE  
Byte (n*12+2)-(n*12+5): 下一个IFD的偏移量，如果没有则置为0  
DE的构成：  
Byte 0-1: 此TAG的唯一标识  
Byte 2-3: 数据类型。  
Byte 4-7: 数量。通过类型和数量可以确定存储此TAG的数据需要占据的字节数  
Byte 8-11: 如果占用的字节数少于4， 则数据直接存于此。 如果超过4个，则这里存放的是指向实际数据的指针

可以用以下的图来表示(图来自http://www.cppblog.com/windcsn/archive/2009/03/12/1158.html)

![](https://img2018.cnblogs.com/blog/1520559/201811/1520559-20181127055336466-69077883.png)

![](https://img2018.cnblogs.com/blog/1520559/201811/1520559-20181127055541721-704308786.png)

![](https://img2018.cnblogs.com/blog/1520559/201811/1520559-20181127055826663-1491985505.png)

在TIFF6.0中，定义了12种数据类型，分别是：

1 = BYTE 8-bit unsigned integer.  
2 = ASCII 8-bit byte that contains a 7-bit ASCII code; the last byte  
must be NUL (binary zero).  
3 = SHORT 16-bit (2-byte) unsigned integer.  
4 = LONG 32-bit (4-byte) unsigned integer.  
5 = RATIONAL Two LONGs: the first represents the numerator  
6 = SBYTE An 8-bit signed (twos-complement) integer.  
7 = UNDEFINED An 8-bit byte that may contain anything, depending on  
the definition of the field.  
8 = SSHORT A 16-bit (2-byte) signed (twos-complement) integer.  
9 = SLONG A 32-bit (4-byte) signed (twos-complement) integer.  
10 = SRATIONAL Two SLONG’s: the first represents the numerator of a  
fraction, the second the denominator.  
11 = FLOAT Single precision (4-byte) IEEE format.  
12 = DOUBLE Double precision (8-byte) IEEE format.

－个TIFF文件可能包含多个IFD，每一个IFD都是一个子文件。Baseline解码器只要求解第一个IFD所对应的图像数据。扩展的TIFF图像经常包含多个IFD，每一个IFD都包含了不同的信息。

TIF图一般由三个部分组成：文件头（简称IFH）、文件目录（简称IFD）、图像数据。  
一、图像文件头（Image File Header）  
　　IFH数据结构包含3个成员共计8个字节（见表一）：  
表一 IFH结构描述  
------------------------------------------------------------  
名称　　　　　　　　字节数　数据类型 说明  
------------------------------------------------------------  
Byteorder　　　　　　　2　　Integer　　TIF标记，其值为4D4D或4949  
Version　　　　　　　　2　　Integer　　版本号，其值恒为2A00  
Offset to first IFD　　4　　Long　　　 第一个IFD的偏移量  
------------------------------------------------------------  
表一说明  
　　1.Byteorder：可能是H4D4D或H4949，H4D4D表示该图是摩托罗拉整数格式，H4949表示该图是Intel整数格式。  
　　2.Version：总是H2A00，它可能是tif文件的版本，也可能用于进一步校验该文件是否为TIF格式。  
　　3.Offset to first IFD：第一个IFD相对文件开始处的偏移量（因为可能会有多个顺序排列的IFD）。  
　　IFD数据结构并不一定紧跟在IFH后面，相反，它常常位于第三部分图像数据的后面，即TIF图像文件的一般组织形式是：IFH——图像数据——IFD。  
　　二、图像文件目录（Image File Directory）  
　　IFD是TIF图像文件中重要的数据结构，它包含了三个成员。由于一个TIF文件中可以有多个图像，而一个IFD只标识一个图像的所有属性（有的文章把“属性”称之为“标签”），所以，一个TIF文件中有几个图像，就会有几个IFD。IFD的结构见表二：  
表二 IFD结构描述  
-----------------------------------------------------------------  
名称　　　　　　　　　字节数　数据类型 说明  
-----------------------------------------------------------------  
Directory Entry Count 　2　　Integer　　本IFD中DE的数量  
Directory Entry(1)　　　12　　　　　　　简称DE，中文译义“目录项”  
Directory Entry(2)　　　12  
……  
Directory Entry(N)　　　12  
Offset to next IFD　　　4　　Long　　　　下一个IFD的偏移量  
-----------------------------------------------------------------  
表二说明  
　　1.Directory Entry Count：指出在该IFD中DE的个数；  
　　2.Directory Entry：共12个字节，结构见表三。需要指出的是，DE的个数是不定的，因为每个DE只标识了图像的一个属性，那么这幅图像有N个属性就会有N个DE，用户甚至可添加自定义的标记属性，这就是为什么称TIF格式文件为“可扩充标记的文件”的原因。  
　　3.Offset to next IFD Or NULL：下一个IFD相对于文件开始处的位置，这是一个链式构成。如果该数字为0，表示已经是最后一个IFD。当然，如果该TIF文件只包含了一幅图像，那么就只有一个IFD，显然这个偏移量也会等于0。  
表三 DE结构描述  
--------------------------------------------------  
名称　　　　　字节数　　数据类型 说明  
--------------------------------------------------  
tag　　　　　　　2　　　Integer　　本属性的标签编号  
type　　　　　　 2　　　Integer　　本属性值的数据类型  
length　　　　　 4　　　Long　　　 该类型数据的数量  
valueOffset　　　4　　　Long　　　 属性值的存放偏移量  
--------------------------------------------------  
表三说明  
　　由DE标识的图像属性有：图像的大小、分辨率、是否压缩、像素的行列数、颜色深度（单色、16色、256色、真彩色）等等。其中：  
　　①tag：是该属性的标签编号（TagID），在图像文件目录中，它是按照升序排列的（但不一定是连续的）。这些编号在TIF格式官方白皮书中可以查到相应的含义，但遗憾的是，我们到哪儿可以找到官方白皮书呢？所以，笔者只能把网上能找得到资料（再结合自己的实验结果）罗列出来，见表四。  
　　②type：表示该属性数据的类型，一般认为TIF官方指定的有5种数据类型（但也有说12种数据类型的）。见表五。  
　　③length：该种类型的数据的个数，而不是某个数据的长度。  
　　④valueOffset：是tagID代表的变量值相对文件开始处的偏移量，但如果变量值占用的空间不多于4个字节（例如只有1个Integer类型的值），那么该值就直接存放在valueOffset中，没必要再另外指向一个地方了。  
表四 DE中标签编号的含义  
-------------------------------------------------------------------------  
TagID　属性名称 type 说明  
-------------------------------------------------------------------------  
0100 图像宽　　　　　　　0003  
0101 图像高　　　　　　　0003  
0102 颜色深度　　　　　　0003　　值＝1为单色，＝4为16色，＝8为256色。  
　　　　　　　　　　　　　 　　　如果该类型数据个数＞2个，说明是真彩图像  
0103 图像数据是否压缩　　0003　　值＝05表示压缩  
0106 图像是否采用反色显示0003　　值＝01表示反色，否则表示不反色  
0111 图像扫描线偏移量　　0004　　图像数据起始字节相对于文件开始处的位置  
0116 图像扫描线的数量　　0004　　表示图像有几行扫描线，实际上等于图像高度  
0117 图像数据字节总数　　0003　　如果不是偶数，那么实际存放时会在后面加0  
011A 水平分辩率偏移量　　0005　　常用计量单位是：像素/英寸  
011B 垂直分辩率 偏移量 　0005　　常用计量单位是：像素/英寸  
0131 生成该图像的软件名　0002　　文本类型  
0132 生成该图像的时间　　0002　　文本类型  
0140 调色板偏移量　　　　0003　　256色和16色图像才有此属性，而且有连续2个  
　　　　　　　　　　　　　　　　 调色板，但属性的length值只表示出1个调色板  
-------------------------------------------------------------------------  
表四说明  
　　①“水平（垂直）分辩率”是分数型的属性，其值要占用8个字节，所以在valueOffset中存放的肯定是它的具体数值的偏移量，而不是数值本身。  
　　②“生成图像的软件名称”和“生成图像的时间”这两个字符型属性，它们的值所占用的空间也会大于4字节，所以在valueOffset中存放的也是它们的值的偏移量，而不是值本身。  
　　③“图像数据字节总数”一般是个偶数，如果是奇数，那么实际存放时会在后面加一个0，但这个0不会计算在字节总数之内。  
表五 DE中的数据类型  
--------------------------------------------------------------------  
type值　数据类型　 说明  
--------------------------------------------------------------------  
0001　　Byte  
0002　　Ascii　　　文本类型，7位Ascii码加1位二进制0  
0003　　Integer  
0004　　Long  
0005　　RATIONAL　 分数类型，由两个Long组成，第1个是分子，第2个是分母  
--------------------------------------------------------------------  
　　三、图像数据。这些数据可能是压缩的，也可能是未压缩的。如果经过压缩，那么压缩算法又有许多种，所以，图像数据是TIF文件中最为复杂的部分，暂还没有哪个软件能译出所有的压缩算法。

TIFF图片可以存储坐标信息，但是如何对这些坐标信息进行转换，然后生成新的图片呢？首先需要知道现在TIFF图片中的坐标系，然后要知道被转换的坐标系。现在我们只支持WGS84、西安80、北京54等中国常用的坐标系。  
坐标系转换的头文件：

头文件

![复制代码](https://common.cnblogs.com/images/copycode.gif)

#pragma once
#include "proj_api.h"

#define PROJ4_COUNT 1024
#define coord_wgs84 "+proj=longlat +ellps=WGS84 +datum=WGS84 +no_defs"
#define coord_google "+proj=merc +a=6378137 +b=6378137 +lat_ts=0.0 +lon_0=0.0 +x_0=0.0 +y_0=0 +k=1.0 +units=m +nadgrids=@null +wktext +no_defs"
/*
北京54 +proj=tmerc +a=6378245.000000 +b=6356863.018800 +x_0=39500000.000000 +y_0=0.000000 +lat_0=0.000000 +lon_0=117.000000 +units=m +k=1.000000  +towgs84=0,-0,0,0,0,0,0
西安80 +proj=tmerc +a=6378140.000000 +b=6356755.288158 +x_0=39500000.000000 +y_0=0.000000 +lat_0=0.000000 +lon_0=117.000000 +units=m +k=1.000000  +towgs84=0,-0,0,0,0,0,0
国家2000 +proj=tmerc +a=6378137.000000 +b=6356752.314140 +x_0=39500000.000000 +y_0=0.000000 +lat_0=0.000000 +lon_0=117.000000 +units=m +k=1.000000  +towgs84=0,-0,0,0,0,0,0
*/

class CoordTrans
{
public:
    CoordTrans();
public:

    void define_coord_sys( const int coord_sys );
    void define_ellps( const int ellps );
    void define_proj_zone( const int maj_zone );
    void define_7_trans( double dx , double dy , double dz , double rx , double ry , double rz , double k );
    void trans_2_des( double&left , double&top , double&right , double&bottom );
    void trans_2_des( double&left , double&top );

    void trans_back_src( double&left , double&top );
public:

    bool init_proj( const char* src_ ,const char* des_ = coord_google );

    void xy_2_latlong( double&x , double&y );
    void xy_2_xy( double&x , double&y );
    void longlat_2_xy( double&x , double&y );

private:
    void deinit();
private:
    void* _source;
    void* _des;

    char* _sz_source;
    char* _sz_des;

private:
    int _coord_sys;
    int _ellps;
    int _maj_zone;

    double _dx;
    double _dy;
    double _dz;
    double _rx;
    double _ry;
    double _rz;
    double _k;
};

![复制代码](https://common.cnblogs.com/images/copycode.gif)

坐标转换cpp文件

![复制代码](https://common.cnblogs.com/images/copycode.gif)

#include "coord_trans.h"
#include <string.h>
#include <stdio.h>
#include "windows.h"
#include <string>
using namespace std;

struct _ellps_type_data_
{
    TCHAR* name;
    double a;
    double b;
};

const _ellps_type_data_ ellps_list[]=
{
    {L"WGS84"   , 6378137.0 , 6356752.3142 },
    {L"西安80"    , 6378140.0 , 6356755.288157528},
    {L"北京54"    , 6378245.0 , 6356863.0188 },
    {L"国家2000", 6378137.0 , 6356752.31414},
    {L"web墨卡托",6378137.0 , 6378137.0},
    {NULL           , 0         , 0 }
};

typedef struct _proj_type_data_
{
    TCHAR* name;
    int zome_number;
    double pl;
    double pb;
    double pfe;
    double pfn;
    double pk;
} ProjData;

ProjData theProj[]=
{
    {L"(6度带)13-中央经线 75"   ,13,75,0,13500000,0,1},
    {L"(6度带)14-中央经线 81"   ,14,81,0,14500000,0,1},
    {L"(6度带)15-中央经线 87"   ,15,87,0,15500000,0,1},
    {L"(6度带)16-中央经线 93"   ,16,93,0,16500000,0,1},
    {L"(6度带)17-中央经线 99" ,17,99,0,17500000,0,1},
    {L"(6度带)18-中央经线 105"    ,18,105,0,18500000,0,1},
    {L"(6度带)19-中央经线 111"    ,19,111,0,19500000,0,1},
    {L"(6度带)20-中央经线 117"    ,20,117,0,20500000,0,1},
    {L"(6度带)21-中央经线 123"    ,21,123,0,21500000,0,1},
    {L"(6度带)22-中央经线 129"    ,22,129,0,22500000,0,1},
    {L"(6度带)23-中央经线 135"    ,23,135,0,23500000,0,1},

    {L"(3度带)25-中央经线 75" ,25,75,0,25500000,0,1},
    {L"(3度带)26-中央经线 78" ,26,78,0,26500000,0,1},
    {L"(3度带)27-中央经线 81" ,27,81,0,27500000,0,1},
    {L"(3度带)28-中央经线 84" ,28,84,0,28500000,0,1},
    {L"(3度带)29-中央经线 87" ,29,87,0,29500000,0,1},
    {L"(3度带)30-中央经线 90" ,30,90,0,30500000,0,1},
    {L"(3度带)31-中央经线 93" ,31,93,0,31500000,0,1},
    {L"(3度带)32-中央经线 96" ,32,96,0,32500000,0,1},
    {L"(3度带)33-中央经线 99" ,33,99,0,33500000,0,1},
    {L"(3度带)34-中央经线 102"    ,34,102,0,34500000,0,1},
    {L"(3度带)35-中央经线 105"    ,35,105,0,35500000,0,1},
    {L"(3度带)36-中央经线 108"    ,36,108,0,36500000,0,1},
    {L"(3度带)37-中央经线 111"    ,37,111,0,37500000,0,1},
    {L"(3度带)38-中央经线 114"    ,38,114,0,38500000,0,1},
    {L"(3度带)39-中央经线 117"    ,39,117,0,39500000,0,1},
    {L"(3度带)40-中央经线 120"    ,40,120,0,40500000,0,1},
    {L"(3度带)41-中央经线 123"    ,41,123,0,41500000,0,1},
    {L"(3度带)42-中央经线 126"    ,42,126,0,42500000,0,1},
    {L"(3度带)43-中央经线 129"    ,43,129,0,43500000,0,1},
    {L"(3度带)44-中央经线 132"    ,44,132,0,44500000,0,1},
    {L"(3度带)45-中央经线 135"    ,45,135,0,45500000,0,1},
    /*{L"自定义",39,117,0,500000,0,1},*/
    {NULL   ,0,0,0,0,1}
};

CoordTrans::CoordTrans()
{
    _source = NULL;
    _des = NULL;

    _sz_source = new char[PROJ4_COUNT];
    memset( _sz_source , 0 , PROJ4_COUNT );

    _sz_des = new char[PROJ4_COUNT];
    memset( _sz_des , 0 , PROJ4_COUNT );
}

void CoordTrans::define_coord_sys( const int coord_sys )
{
    _coord_sys = coord_sys;

}

void CoordTrans::define_ellps( const int ellps )
{
    _ellps = ellps;
}

void CoordTrans::define_proj_zone( const int maj_zone )
{
    _maj_zone = maj_zone;
}
void CoordTrans::define_7_trans( double dx , double dy , double dz , double rx , double ry , double rz , double k )
{
    _dx = dx;
    _dy = -dy;
    _dz = dz;
    _rx = rx;
    _ry = ry;
    _rz = rz;
    _k = k;
}

void CoordTrans::trans_2_des( double&left , double&top , double&right , double&bottom )
{
    double left2 = left;
    double right2 = right;
    double top2 = top;
    double bottom2 = bottom;

    string proj ;
    if ( _coord_sys == 0 )
    {
        proj = "+proj=longlat ";
    }
    else if ( _coord_sys == 1 )
    {
        proj = "+proj=tmerc ";
    }

    //椭球体
    char ch_ellps[128];
    memset( ch_ellps , 0 , 128 );
    sprintf( ch_ellps , "+a=%f +b=%f " , ellps_list[_ellps].a , ellps_list[_ellps].b );
    string ellps( ch_ellps );

    //投影带
    char ch_zone[512];
    memset( ch_zone , 0 , 512 );
    sprintf ( ch_zone ,"+x_0=%f +y_0=0.000000 +lat_0=0.000000 +lon_0=%f +units=m +k=1.000000 "
        , theProj[_maj_zone].pfe , theProj[_maj_zone].pl );
    string zone( ch_zone );

    //towgs84
    char ch_wgs84[128];
    memset( ch_wgs84 , 0 , 128 );
    sprintf( ch_wgs84 , "+towgs84=%f,%f,%f,%f,%f,%f,%f" , _dx , _dy , _dz , _rx , _ry , _rz , _k );
    string towgs84( ch_wgs84 );

    sprintf ( _sz_source ,"%s%s%s%s" , proj.c_str() , ellps.c_str() , zone.c_str() , towgs84.c_str() );


    init_proj( _sz_source );

    if ( _coord_sys == 0 )
    {
        longlat_2_xy( left , top );
        longlat_2_xy( right , bottom );
    }
    else
    {
        xy_2_xy( left , top );
        xy_2_xy( right , bottom );
    }


    /*init_proj( _sz_source , coord_wgs84 );
    xy_2_latlong( left2 , top2 );
    xy_2_latlong( right2 , bottom2 );
    init_proj( coord_wgs84 , coord_google );
    longlat_2_xy( left2 , top2 );
    longlat_2_xy( right2 , bottom2  );*/
}

void CoordTrans::trans_2_des( double&left , double&top )
{
    double left2 = left;
    double top2 = top;

    string proj ;
    if ( _coord_sys == 0 )
    {
        proj = "+proj=longlat ";
    }
    else if ( _coord_sys == 1 )
    {
        proj = "+proj=tmerc ";
    }

    //椭球体
    char ch_ellps[128];
    memset( ch_ellps , 0 , 128 );
    sprintf( ch_ellps , "+a=%f +b=%f " , ellps_list[_ellps].a , ellps_list[_ellps].b );
    string ellps( ch_ellps );

    //投影带
    char ch_zone[512];
    memset( ch_zone , 0 , 512 );
    sprintf ( ch_zone ,"+x_0=%f +y_0=0.000000 +lat_0=0.000000 +lon_0=%f +units=m +k=1.000000 "
        , theProj[_maj_zone].pfe , theProj[_maj_zone].pl );
    string zone( ch_zone );

    //towgs84
    char ch_wgs84[128];
    memset( ch_wgs84 , 0 , 128 );
    sprintf( ch_wgs84 , "+towgs84=%f,%f,%f,%f,%f,%f,%f" , _dx , _dy , _dz , _rx , _ry , _rz , _k );
    string towgs84( ch_wgs84 );

    sprintf ( _sz_source ,"%s%s%s%s" , proj.c_str() , ellps.c_str() , zone.c_str() , towgs84.c_str() );


    init_proj( _sz_source );

    if ( _coord_sys == 0 )
    {
        longlat_2_xy( left , top );
    }
    else
    {
        xy_2_xy( left , top );
    }
}

void CoordTrans::trans_back_src( double&left , double&top )
{
    double left2 = left;
    double top2 = top;

    string proj ;
    if ( _coord_sys == 0 )
    {
        proj = "+proj=longlat ";
    }
    else if ( _coord_sys == 1 )
    {
        proj = "+proj=tmerc ";
    }

    //椭球体
    char ch_ellps[128];
    memset( ch_ellps , 0 , 128 );
    sprintf( ch_ellps , "+a=%f +b=%f " , ellps_list[_ellps].a , ellps_list[_ellps].b );
    string ellps( ch_ellps );

    //投影带
    char ch_zone[512];
    memset( ch_zone , 0 , 512 );
    sprintf ( ch_zone ,"+x_0=%f +y_0=0.000000 +lat_0=0.000000 +lon_0=%f +units=m +k=1.000000 "
        , theProj[_maj_zone].pfe , theProj[_maj_zone].pl );
    string zone( ch_zone );

    //towgs84
    char ch_wgs84[128];
    memset( ch_wgs84 , 0 , 128 );
    sprintf( ch_wgs84 , "+towgs84=%f,%f,%f,%f,%f,%f,%f" , _dx , _dy , _dz , _rx , _ry , _rz , _k );
    string towgs84( ch_wgs84 );

    sprintf ( _sz_des ,"%s%s%s%s" , proj.c_str() , ellps.c_str() , zone.c_str() , towgs84.c_str() );


    init_proj( coord_google , _sz_des );

    if ( _coord_sys == 0 )
    {
        longlat_2_xy( left , top );
    }
    else
    {
        xy_2_xy( left , top );
    }
}

bool CoordTrans::init_proj( const char* src_ ,const char* des_ )
{
    deinit();

    _source = pj_init_plus( src_ );
    if ( !_source )
    {
        return false;
    }
    _des = pj_init_plus( des_ );
    if ( !_des )
    {
        return false;
    }

    return true;
}

void CoordTrans::xy_2_latlong( double&x , double&y )
{
    pj_transform(  _source,_des, 1, 0, &x , &y , 0 );

    x *= RAD_TO_DEG;
    y *= RAD_TO_DEG;
}

void CoordTrans::xy_2_xy( double&x , double&y )
{
    pj_transform(  _source,_des, 1, 0, &x , &y , 0 );
}
void CoordTrans::longlat_2_xy( double&x , double&y )
{
    x *= DEG_TO_RAD;
    y *= DEG_TO_RAD;
    pj_transform(  _source ,_des ,  1, 0, &x , &y , 0 );

}

void CoordTrans::deinit()
{
    if ( _source )
    {
        pj_free( _source );
        _source = NULL;
    }

    if ( _des )
    {
        pj_free( _des );

        _des= NULL;
    }
}

![复制代码](https://common.cnblogs.com/images/copycode.gif)

tiff处理的头文件

![复制代码](https://common.cnblogs.com/images/copycode.gif)

#pragma once
#include "tiflib.h"
#include "coord_trans.h"
#include <math.h>
#include <string>
#include <vector>
using namespace std;

#include<Windows.h>

#define TIFF_HEADER_SIZE 8    //文件头字节数
#define DE_START 10           //tiff TAG开始的位置
#define ONE_DE_SIZE 12        //每个TAG的大小
#define IDF_END_FLAG_SIZE 4   //IFD最后结尾的4个空字节
typedef struct
{
    int i_tag;
    const char* text;
}TagText;

typedef struct
{
    TIFF_UINT16_T type;
    char* type_name;
    TIFF_UINT16_T type_size;
}DataType;
typedef struct
{
    TIFF_UINT16_T tag;
    TIFF_UINT16_T type;
    TIFF_UINT32_T count;
    TIFF_UINT32_T offset;
}DirectoryEntry;

typedef struct
{
    DirectoryEntry de;
    int data_source; //0 - offset本身值 1 - offset对应的源文件偏移量 2 - 来自内存
    TIFF_UINT8_T* mem_data; //当 data_soure = 2 时 ，指向内存
}deInfo;

typedef struct
{
    int diff_x ;
    int diff_y ;
}DiffValue;

class tiffDeform
{
public:
    tiffDeform();
    tiffDeform( string tiff_path , string proj4_str );
    ~tiffDeform();

public:
    int open( string tiff_path , string proj4_str  , int coord_type );
    bool arrangement();
    void save( string src_file , string save_name = "" );
private:
    void src_coord_box( );
    double zone_number( double left_right );
    int get_zone_num( string proj4_str );
    double src_geo_xy( double left_right );
    void to_web_mector( geoRECT&rect , bool b );//b - true 取最内侧的范围 false - 取最外侧范围
    void to_web_mector( geoRECT&rect , bool b , double&x1 , double&y1, double&x2 , double&y2, double&x3 , double&y3, double&x4 , double&y4 );
    void get_pixel_scale();
    int get_src_tag_list();
    int cts_new_tag_list();
    void print_tag_info_list();
    const char* tag_text( int i_tag );

    deInfo* cts_strip_offsets();  
    deInfo* cts_rows_per_strip();
    deInfo* cts_strip_byte_counts();
    deInfo* cts_line_tag();
    void mix_proj_de_list( deInfo* coord_list );

    void write_file_header( );
    string new_tiff_name();
    void modify_strip_offset();
    void sort_byte_order( TIFF_UINT8_T* buf , TIFF_UINT16_T data_type , TIFF_UINT16_T data_count );
    TIFF_UINT64_T file_disk_data( DirectoryEntry de , TIFF_UINT64_T buffer_size );
    void write_tag_list();

    void write_img_data();       //单个像素写入
    void write_img_data_ex();    //提高写入像素的速度,整行写入
    void write_img_data_th();    //采用多行读取当行写入来提高效率
    void write_img_data_th_ex(); //采用多行读取当行写入来提高效率,并且记录TXT像素差值
    void write_img_data_mul_th();//采用多线程
    void write_img_by_record();  //通过事先记录好的规则填写数据
    void write_img_by_block_record();//分块计算规则
    void write_img_by_sin();     //通过斜率填写数据
    TIFF_UINT64_T double_to_long( double d_ );
    void point_to_by_geo( double x , double y , int&pt_x , int&pt_y );
    void point_to_by_geo_ex( double x , double y , int&pt_x , int&pt_y );

    void cs_coord( deInfo* coord_list );
    void cs_tie_point( double x , double y );
    void cs_pixel_scale( double cx , double cy );

    void des_geo_to_src_range( geoRECT des_geo , geoRECT&src_range );//通过目标地理范围获得原图片的像素范围
    void to_src_proj( geoRECT&des_geo , bool b );//b - true 取最内侧的范围 false - 取最外侧范围

    void des_geo_to_src_range_first( geoRECT des_geo , geoRECT&src_range , int&first_x , int&first_y );//通过目标地理范围获得原图片的像素范围,并得到第一个点坐标值
    void to_src_proj_first( geoRECT&des_geo , bool b , double&first_x , double&first_y );//b - true 取最内侧的范围 false - 取最外侧范围

private:
    HANDLE m_htiff_write;
    static DWORD WINAPI tiff_write_thread( LPVOID lpParameter );

private:
    TiffFile* _tiff_file;
    string _tiff_path;
    string _proj4_str;

    CoordTrans _coord_trans;
    int _coord_type;
    geoRECT _geo_rect_src , _geo_rect_des; //图片的边界
    double _scaleX_src,_scaleY_src;
    double _scaleX_des,_scaleY_des;

    int _new_tiff_width;
    int _new_tiff_height;

    FILE* _line_tiff;
    deInfo* de_list;
    TIFF_UINT16_T _de_num;             //标签的数量
    TIFF_UINT32_T _strip_offset_pos;   //TAG StripOffset的文件偏移位置

    TIFF_UINT64_T pixel_scale[3];//像元比例
    TIFF_UINT64_T tie_point[6];//控制点坐标对
    TIFF_UINT32_T* _height_start;
    TIFF_UINT32_T _file_size;

    HANDLE _hMutex;
};

![复制代码](https://common.cnblogs.com/images/copycode.gif)

tiff处理的cpp

![复制代码](https://common.cnblogs.com/images/copycode.gif)

#include "tiffDeform.h"

TagText tag_text_list[] =
{
    { 254 , "NewSubfileType" },
    { 256 , "ImageWidth" },
    { 257 , "ImageLength" },
    { 258 , "BitsPerSample" },
    { 259 , "Compression" },
    { 262 , "PhotometricInterpretation" },
    { 273 , "StripOffsets" },
    { 274 , "相对于图像的行和列的方向" },
    { 277 , "SamplesPerPixel" },
    { 278 , "RowsPerStrip" },
    { 279 , "StripByteCounts" },
    { 282 , "XResolution" },
    { 283 , "YResolution" },
    { 284 , "PlanarConfiguration" },
    { 296 , "ResolutionUnit" },
    { 305 , "Software" },
    { 306 , "DateTime" },
    { 322 , "TileWidth" },
    { 323 , "TileLength" },
    { 324 , "TileOffsets" },
    { 325 , "TileByteCounts" },
    { 339 , "SampleFormat" },
    { 33550 , "ModelPixelScaleTag" },
    { 33922 , "ModelTiepointTag" },
    { 34264 , "ModelTransformationTag" },
    { 34735 , "GeoKeyDirectoryTag" },
    { 34736 , "GeoDoubleParamsTag" },
    { 34737 , "GeoAsciiParamsTag" },
    { -1 , "" }
};

DataType data_type_list[] =
{
    { 0  , "NULL"     , 0 },//NULL
    { 1  , "BYTE"     , 1 },//BYTE 8-bit unsigned integer
    { 2  , "ASCII"    , 1 },//ASCII 8-bit byte that contains a 7-bit ASCII code; the last byte must be NUL (binary zero)
    { 3  , "SHORT"    , 2 },//SHORT 16-bit (2-byte) unsigned integer
    { 4  , "LONG"     , 4 },//LONG 32-bit (4-byte) unsigned integer
    { 5  , "RATIONAL" , 8 },//RATIONAL Two LONGs: the first represents the numerator
    { 6  , "SBYTE"    , 1 },//SBYTE An 8-bit signed (twos-complement) integer.
    { 7  , "UNDEFINED", 1 },//UNDEFINED An 8-bit byte that may contain anything, depending on the definition of the field.
    { 8  , "SSHORT"   , 2 },//SSHORT A 16-bit (2-byte) signed (twos-complement) integer
    { 9  , "SLONG"    , 4 },//SLONG A 32-bit (4-byte) signed (twos-complement) integer
    { 10 , "SRATIONAL", 8 },//SRATIONAL Two SLONG’s: the first represents the numerator of afraction, the second the denominator
    { 11 , "FLOAT"    , 4 },//FLOAT Single precision (4-byte) IEEE format
    { 12 , "DOUBLE"   , 8 } //DOUBLE Double precision (8-byte) IEEE format.
};

/*
Geo Key : 1024 = 1 
Geo Key : 1025 = 1 
Geo Key : 1026 = Popular Visualisation CRS / Mercator 
Geo Key : 2048 = 32767 
Geo Key : 2049 = Popular Visualisation CRS 
Geo Key : 2050 = 32767 
Geo Key : 2054 = 9102 
Geo Key : 2056 = 32767 
Geo Key : 2057 = 6378137.000000 
Geo Key : 2058 = 6378137.000000 
Geo Key : 3072 = 32767 
Geo Key : 3074 = 32767 
Geo Key : 3075 = 7 
Geo Key : 3076 = 9001 
Geo Key : 3080 = 0.000000 
Geo Key : 3081 = 0.000000 
Geo Key : 3082 = 0.000000 
Geo Key : 3083 = 0.000000 
Geo Key : 3092 = 1.000000 
*/
unsigned char geotiff_web_mercator_tag_dir[]=
{
    /*目录版本号(2 bytes),修订版本号(2 bytes),副版本号(2 bytes),地理键的个数(2 bytes)*/
    0x01, 0x00, 0x01, 0x00, 0x00, 0x00, 0x13, 0x00, //
    /*地理键ID(2 bytes),存放位置(2 bytes),元素的个数(2 bytes),值/索引(2 bytes) */
    0x00, 0x04, 0x00, 0x00, 0x01, 0x00, 0x01, 0x00, 
    0x01, 0x04, 0x00, 0x00, 0x01, 0x00, 0x01, 0x00, 
    0x02, 0x04, 0xb1, 0x87, 0x25, 0x00, 0x00, 0x00, 
    0x00, 0x08, 0x00, 0x00, 0x01, 0x00, 0xff, 0x7f, 
    0x01, 0x08, 0xb1, 0x87, 0x1a, 0x00, 0x25, 0x00,
    0x02, 0x08, 0x00, 0x00, 0x01, 0x00, 0xff, 0x7f, 
    0x06, 0x08, 0x00, 0x00, 0x01, 0x00, 0x8e, 0x23, 
    0x08, 0x08, 0x00, 0x00, 0x01, 0x00, 0xff, 0x7f, 
    0x09, 0x08, 0xb0, 0x87, 0x01, 0x00, 0x05, 0x00, 
    0x0a, 0x08, 0xb0, 0x87, 0x01, 0x00, 0x06, 0x00, 
    0x00, 0x0c, 0x00, 0x00, 0x01, 0x00, 0xff, 0x7f, 
    0x02, 0x0c, 0x00, 0x00, 0x01, 0x00, 0xff, 0x7f,
    0x03, 0x0c, 0x00, 0x00, 0x01, 0x00, 0x07, 0x00,
    0x04, 0x0c, 0x00, 0x00, 0x01, 0x00, 0x29, 0x23,
    0x08, 0x0c, 0xb0, 0x87, 0x01, 0x00, 0x01, 0x00, 
    0x09, 0x0c, 0xb0, 0x87, 0x01, 0x00, 0x00, 0x00, 
    0x0a, 0x0c, 0xb0, 0x87, 0x01, 0x00, 0x03, 0x00, 
    0x0b, 0x0c, 0xb0, 0x87, 0x01, 0x00, 0x04, 0x00,
    0x14, 0x0c, 0xb0, 0x87, 0x01, 0x00, 0x02, 0x00
};

unsigned char geotiff_double_param[]=
{
    0x00 , 0x00 , 0x00 , 0x00 , 0x00 , 0x00 , 0x00 , 0x00 , //0.0
    0x00 , 0x00 , 0x00 , 0x00 , 0x00 , 0x00 , 0x00 , 0x00 , //0.0
    0x00 , 0x00 , 0x00 , 0x00 , 0x00 , 0x00 , 0xf0 , 0x3f , //1.0
    0x00 , 0x00 , 0x00 , 0x00 , 0x00 , 0x00 , 0x00 , 0x00 , //0.0
    0x00 , 0x00 , 0x00 , 0x00 , 0x00 , 0x00 , 0x00 , 0x00 , //0.0
    0x00 , 0x00 , 0x00 , 0x40 , 0xa6 , 0x54 , 0x58 , 0x41 , //6378137.0000000000
    0x00 , 0x00 , 0x00 , 0x40 , 0xa6 , 0x54 , 0x58 , 0x41   //6378137.0000000000
};

unsigned char geotiff_ascii_param[]= /*Popular Visualisation CRS / Mercator|Popular Visualisation CRS|*/
{
    0x50 , 0x6f , 0x70 , 0x75 , 0x6c , 0x61 , 0x72 , 0x20 , 
    0x56 , 0x69 , 0x73 , 0x75 , 0x61 , 0x6c , 0x69 , 0x73 , 
    0x61 , 0x74 , 0x69 , 0x6f , 0x6e , 0x20 , 0x43 , 0x52 , 
    0x53 , 0x20 , 0x2f , 0x20 , 0x4d , 0x65 , 0x72 , 0x63 , 
    0x61 , 0x74 , 0x6f , 0x72 , 0x7c , 0x50 , 0x6f , 0x70 , 
    0x75 , 0x6c , 0x61 , 0x72 , 0x20 , 0x56 , 0x69 , 0x73 , 
    0x75 , 0x61 , 0x6c , 0x69 , 0x73 , 0x61 , 0x74 , 0x69 , 
    0x6f , 0x6e , 0x20 , 0x43 , 0x52 , 0x53 , 0x7c , 0x00
};

tiffDeform::tiffDeform()
{
    _tiff_file = NULL;
    _tiff_file = new TiffFile;
    memset( _tiff_file , 0 , sizeof(TiffFile) );
}

tiffDeform::tiffDeform( string tiff_path , string proj4_str )
{
    _tiff_path = tiff_path ;
    _proj4_str = proj4_str ;

    _tiff_file = new TiffFile;
    memset( _tiff_file , 0 , sizeof(TiffFile) );
}

tiffDeform::~tiffDeform()
{
    if( _tiff_file == NULL )
    {
        delete _tiff_file;
        _tiff_file = NULL;
    }
}

bool tiffDeform::arrangement()
{
    return _tiff_file->tile.is_tile;
}

int tiffDeform::open( string tiff_path , string proj4_str , int coord_type )
{
    _tiff_path = tiff_path ;
    _proj4_str = proj4_str ;
    _coord_type = coord_type;
    return tif_open ( tiff_path.c_str() , _tiff_file );
}

void tiffDeform::save(  string src_file , string save_name )
{
    delete _tiff_file;
    _tiff_file = NULL;

    _tiff_path = src_file;
    _tiff_file = new TiffFile;
    memset( _tiff_file , 0 , sizeof(TiffFile) );
    tif_open( _tiff_path.c_str() , _tiff_file );

    //1.获得原始图片的范围坐标
    src_coord_box( );

    //2.计算转换为WebMector的坐标范围
    _geo_rect_des = _geo_rect_src;
    _geo_rect_des.left = zone_number( _geo_rect_des.left ) ;
    _geo_rect_des.right = zone_number( _geo_rect_des.right ) ;
    double x1,y1,x2,y2,x3,y3,x4,y4;
    to_web_mector( _geo_rect_des , true ,x1,y1,x2,y2,x3,y3,x4,y4 );

    //3.获得像元比例
    get_pixel_scale();

    //4.计算新生成的图片的长度和宽度
    _new_tiff_width = (int)(( _geo_rect_des.right - _geo_rect_des.left )/_scaleX_des + 0.5 );
    _new_tiff_height= (int)(( _geo_rect_des.top - _geo_rect_des.botton )/_scaleY_des + 0.5 );

    //5.获得原图片的TAG列表
    get_src_tag_list();


    //6.构造新的TAG列表
    cts_new_tag_list();


    string temp_path;
    if (  save_name == "" )
    {
        temp_path = new_tiff_name() ;
    }
    else
    {
        temp_path = save_name;
    }

    _line_tiff = fopen( temp_path.c_str() , "wb" );
    if ( _line_tiff == NULL )
    {
        return ;
    }
    //7.写入文件头
    write_file_header( );

    //8.写入tag的数量
    fwrite( &( _de_num ) , 1 , 2 , _line_tiff );
    //9.写入空的DE占位空间
    TIFF_UINT8_T* place_holder = new TIFF_UINT8_T[ _de_num * ONE_DE_SIZE + IDF_END_FLAG_SIZE ];
    memset( place_holder , 0 , _de_num * ONE_DE_SIZE + IDF_END_FLAG_SIZE );
    fwrite( place_holder , 1 , _de_num * ONE_DE_SIZE + IDF_END_FLAG_SIZE , _line_tiff );

    TIFF_UINT64_T write_file_size = ftell( _line_tiff );
    //10.写入具体的TAG内容和对应的偏移量
    write_tag_list();

    //11.修改图像数据的偏移量
    modify_strip_offset();

    //12.写入图像数据
    //write_img_data();
    //write_img_data_ex();
    //write_img_data_th();
    //write_img_data_th_ex();
    //write_img_data_mul_th( );
    write_img_by_record();
    //write_img_by_sin();
    //write_img_by_block_record();

    //13.关闭文件
    //Sleep( 1000 );
    fclose( _line_tiff );
}

void tiffDeform::write_img_data()
{
    TIFF_UINT32_T* height_start = new TIFF_UINT32_T[ _new_tiff_height *  sizeof(TIFF_UINT32_T) ];
    memset( height_start , 0 ,  _new_tiff_height *  sizeof(TIFF_UINT32_T) );
    int bits =  _tiff_file->bit_per_samples ;
    if ( bits >= 24 )
    {
        if ( !_tiff_file->tile.is_tile )//不是瓦片数据
        {
            fresh_line_start( _tiff_file , height_start );
        }
    }

    fseek( _line_tiff , 0 , SEEK_END );

    _coord_trans.init_proj( coord_google , _proj4_str.c_str() );

    for ( int i_height = 0 ; i_height < _new_tiff_height ; i_height++ )
    {
        system("cls");
        printf( " %s \n Coord Trans %d / %d \n" , _tiff_path.c_str() , i_height + 1 , _new_tiff_height  );
        for ( int i_width = 0 ; i_width < _new_tiff_width ; i_width++ )
        {
            double new_y = _geo_rect_des.top;
            new_y -= ( i_height * _scaleY_des ); 
            double new_x = _geo_rect_des.left;
            new_x += ( i_width * _scaleX_des );

            if ( _coord_type == 0 )
            {
                _coord_trans.xy_2_latlong( new_x , new_y );
            }
            else
            {
                _coord_trans.xy_2_xy( new_x , new_y );
            }

            new_x = src_geo_xy( new_x );

            int pixel_x = 0 , pixel_y = 0 ;
            point_to_by_geo( new_x , new_y , pixel_x , pixel_y );

            TIFF_UINT8_T* buf = new TIFF_UINT8_T[ _tiff_file->samples_per_pixel ];
            memset( buf , 0 , _tiff_file->samples_per_pixel );
            if ( !( pixel_y == -1 || pixel_x == -1 ) )
            {   
                TIFF_UINT32_T pos = height_start[pixel_y] + pixel_x*_tiff_file->samples_per_pixel;
                fseek( _tiff_file->pfile , height_start[pixel_y] + pixel_x*_tiff_file->samples_per_pixel , SEEK_SET );
                fread( buf , _tiff_file->samples_per_pixel , 1 , _tiff_file->pfile );

                fwrite( buf , _tiff_file->samples_per_pixel , 1 , _line_tiff );
            }
            else
            {
                fwrite( buf , _tiff_file->samples_per_pixel , 1 , _line_tiff );
            }
            delete[] buf;
            buf = NULL;
        }
    }
}

void tiffDeform::write_img_data_ex()
{
    TIFF_UINT32_T* height_start = new TIFF_UINT32_T[ _new_tiff_height *  sizeof(TIFF_UINT32_T) ];
    memset( height_start , 0 ,  _new_tiff_height *  sizeof(TIFF_UINT32_T) );
    int bits =  _tiff_file->bit_per_samples ;
    if ( bits >= 24 )
    {
        if ( !_tiff_file->tile.is_tile )//不是瓦片数据
        {
            fresh_line_start( _tiff_file , height_start );
        }
    }

    fseek( _line_tiff , 0 , SEEK_END );

    _coord_trans.init_proj( coord_google , _proj4_str.c_str() );

    printf( "转换中请稍后... \n" );

    for ( int i_height = 0 ; i_height < _new_tiff_height ; i_height++ )
    {
        //system("cls");
        //printf( " %s \n Coord Trans %d / %d \n" , _tiff_path.c_str() , i_height + 1 , _new_tiff_height  );
        TIFF_UINT8_T* buf = new TIFF_UINT8_T[ _tiff_file->samples_per_pixel * _new_tiff_width ];
        memset( buf , 0 , _tiff_file->samples_per_pixel * _new_tiff_width );
        TIFF_UINT8_T* temp_buf = buf ;

        for ( int i_width = 0 ; i_width < _new_tiff_width ; i_width++ )
        {
            double new_y = _geo_rect_des.top;
            new_y -= ( i_height * _scaleY_des ); 
            double new_x = _geo_rect_des.left;
            new_x += ( i_width * _scaleX_des );

            if ( _coord_type == 0 )
            {
                _coord_trans.xy_2_latlong( new_x , new_y );
            }
            else
            {
                _coord_trans.xy_2_xy( new_x , new_y );
            }

            new_x = src_geo_xy( new_x );//添加带号

            int pixel_x = 0 , pixel_y = 0 ;
            point_to_by_geo( new_x , new_y , pixel_x , pixel_y );

            //TIFF_UINT8_T* buf = new TIFF_UINT8_T[ _tiff_file->samples_per_pixel ];
            //memset( buf , 0 , _tiff_file->samples_per_pixel );
            if ( !( pixel_y == -1 || pixel_x == -1 ) )
            {   
                TIFF_UINT32_T pos = height_start[pixel_y] + pixel_x*_tiff_file->samples_per_pixel;
                fseek( _tiff_file->pfile , height_start[pixel_y] + pixel_x*_tiff_file->samples_per_pixel , SEEK_SET );
                fread( temp_buf , _tiff_file->samples_per_pixel , 1 , _tiff_file->pfile );

                //fwrite( buf , _tiff_file->samples_per_pixel , 1 , _line_tiff );
                temp_buf += _tiff_file->samples_per_pixel ;
            }
            else
            {
                //fwrite( buf , _tiff_file->samples_per_pixel , 1 , _line_tiff );
                temp_buf += _tiff_file->samples_per_pixel ;
            }
            //delete[] buf;
            //buf = NULL;
        }
        fwrite( buf , _tiff_file->samples_per_pixel * _new_tiff_width , 1 , _line_tiff );
        delete[] buf;
        buf = NULL;
    }

    printf( "转换完成！ \n" );
}

void tiffDeform::to_src_proj_first( geoRECT&des_geo , bool b , double&first_x , double&first_y )
{
/*
      (1)     (2)
        +-----+
        |     |
        |     |
        +-----+
      (3)     (4) 
    */
    //_coord_trans.init_proj( coord_google , _proj4_str.c_str() );
    double x1,y1,x2,y2,x3,y3,x4,y4;
    x1 = des_geo.left;  y1 = des_geo.top;
    x2 = des_geo.right; y2 = des_geo.top;
    x3 = des_geo.left ; y3 = des_geo.botton;
    x4 = des_geo.right; y4 = des_geo.botton;

    if ( _coord_type == 0 )
    {
        _coord_trans.xy_2_latlong( x1 , y1 );
    }
    else
    {
        _coord_trans.xy_2_xy( x1 , y1 );
    }

    first_x = x1 ;
    first_y = y1 ;

    if ( _coord_type == 0 )
    {
        _coord_trans.xy_2_latlong( x2 , y2 );
    }
    else
    {
        _coord_trans.xy_2_xy( x2 , y2 );
    }

    if ( _coord_type == 0 )
    {
        _coord_trans.xy_2_latlong( x3 , y3 );
    }
    else
    {
        _coord_trans.xy_2_xy( x3 , y3 );
    }

    if ( _coord_type == 0 )
    {
        _coord_trans.xy_2_latlong( x4 , y4 );
    }
    else
    {
        _coord_trans.xy_2_xy( x4 , y4 );
    }

    if ( b )
    {
        des_geo.left = x1 < x3 ? x3 : x1;
        des_geo.top = y1 > y2 ? y2 : y1;
        des_geo.right = x2 > x4 ? x4 : x2;
        des_geo.botton = y3 < y4 ? y4 : y3;
    }
    else
    {
        des_geo.left = x1 > x3 ? x3 : x1;
        des_geo.top = y1 < y2 ? y2 : y1;
        des_geo.right = x2 < x4 ? x4 : x2;
        des_geo.botton = y3 > y4 ? y4 : y3;
    }
}

void tiffDeform::to_src_proj( geoRECT&des_geo , bool b )
{
    /*
      (1)     (2)
        +-----+
        |     |
        |     |
        +-----+
      (3)     (4) 
    */
    //_coord_trans.init_proj( coord_google , _proj4_str.c_str() );
    double x1,y1,x2,y2,x3,y3,x4,y4;
    x1 = des_geo.left;  y1 = des_geo.top;
    x2 = des_geo.right; y2 = des_geo.top;
    x3 = des_geo.left ; y3 = des_geo.botton;
    x4 = des_geo.right; y4 = des_geo.botton;

    if ( _coord_type == 0 )
    {
        _coord_trans.xy_2_latlong( x1 , y1 );
    }
    else
    {
        _coord_trans.xy_2_xy( x1 , y1 );
    }

    if ( _coord_type == 0 )
    {
        _coord_trans.xy_2_latlong( x2 , y2 );
    }
    else
    {
        _coord_trans.xy_2_xy( x2 , y2 );
    }

    if ( _coord_type == 0 )
    {
        _coord_trans.xy_2_latlong( x3 , y3 );
    }
    else
    {
        _coord_trans.xy_2_xy( x3 , y3 );
    }

    if ( _coord_type == 0 )
    {
        _coord_trans.xy_2_latlong( x4 , y4 );
    }
    else
    {
        _coord_trans.xy_2_xy( x4 , y4 );
    }

    if ( b )
    {
        des_geo.left = x1 < x3 ? x3 : x1;
        des_geo.top = y1 > y2 ? y2 : y1;
        des_geo.right = x2 > x4 ? x4 : x2;
        des_geo.botton = y3 < y4 ? y4 : y3;
    }
    else
    {
        des_geo.left = x1 > x3 ? x3 : x1;
        des_geo.top = y1 < y2 ? y2 : y1;
        des_geo.right = x2 < x4 ? x4 : x2;
        des_geo.botton = y3 > y4 ? y4 : y3;
    }
}

void tiffDeform::des_geo_to_src_range( geoRECT des_geo , geoRECT&src_range )
{
    to_src_proj( des_geo , false );
    src_range.left = ( des_geo.left - _geo_rect_src.left ) / _scaleX_src - 0.5 ;
    if ( src_range.left < 0 )
    {
        //src_range.left = 0 ;
    }

    src_range.right = ( des_geo.right - _geo_rect_src.left ) / _scaleX_src + 0.5 ;
    if ( src_range.right > _tiff_file->tif_width )
    {
        //src_range.right = _tiff_file->tif_width ;
    }

    src_range.top = ( _geo_rect_src.top - des_geo.top ) / _scaleY_src - 0.5;
    if ( src_range.top < 0 )
    {
        src_range.top = 0;
    }
    src_range.botton = ( _geo_rect_src.top - des_geo.botton ) / _scaleY_src + 0.5;
    if ( src_range.botton > _tiff_file->tif_height )
    {
        src_range.botton = _tiff_file->tif_height ;
    }
}

void tiffDeform::des_geo_to_src_range_first( geoRECT des_geo , geoRECT&src_range , int&first_x , int&first_y )
{
    double fx = 0.0 , fy = 0.0 ;
    to_src_proj_first( des_geo , false , fx , fy );

    first_x = (int)(( fx - _geo_rect_src.left ) / _scaleX_src - 0.5 ) ;
    first_y = (int)(( _geo_rect_src.top - fy ) / _scaleY_src - 0.5); 

    src_range.left = ( des_geo.left - _geo_rect_src.left ) / _scaleX_src - 0.5 ;
    if ( src_range.left < 0 )
    {
        src_range.left = 0 ;
    }

    src_range.right = ( des_geo.right - _geo_rect_src.left ) / _scaleX_src + 0.5 ;
    if ( src_range.right > _tiff_file->tif_width )
    {
        src_range.right = _tiff_file->tif_width ;
    }

    src_range.top = ( _geo_rect_src.top - des_geo.top ) / _scaleY_src - 0.5;
    if ( src_range.top < 0 )
    {
        src_range.top = 0;
    }
    src_range.botton = ( _geo_rect_src.top - des_geo.botton ) / _scaleY_src + 0.5;
    if ( src_range.botton > _tiff_file->tif_height )
    {
        src_range.botton = _tiff_file->tif_height ;
    }
}

void tiffDeform::write_img_data_th()
{
    TIFF_UINT32_T* height_start = new TIFF_UINT32_T[ _new_tiff_height *  sizeof(TIFF_UINT32_T) ];
    memset( height_start , 0 ,  _new_tiff_height *  sizeof(TIFF_UINT32_T) );
    int bits =  _tiff_file->bit_per_samples ;
    if ( bits >= 24 )
    {
        if ( !_tiff_file->tile.is_tile )//不是瓦片数据
        {
            fresh_line_start( _tiff_file , height_start );
        }
    }

    fseek( _line_tiff , 0 , SEEK_END );

    printf( "转换中请稍后... \n" );

    _coord_trans.init_proj( coord_google , _proj4_str.c_str() );

    for ( int i_height = 0 ; i_height < _new_tiff_height ; i_height++ )
    {
        double line_y = _geo_rect_des.top;
        line_y -= ( i_height * _scaleY_des ); 

        geoRECT new_line_rect ;
        new_line_rect.left = _geo_rect_des.left ;
        new_line_rect.right = _geo_rect_des.right ;
        new_line_rect.top = line_y ;
        new_line_rect.botton = line_y ;
        geoRECT src_range;
        des_geo_to_src_range( new_line_rect , src_range );

        int src_top = ( int )src_range.top;
        int src_bottom = ( int )src_range.botton;
        TIFF_UINT32_T src_buf_size = _tiff_file->samples_per_pixel * _tiff_file->tif_width * ( src_bottom - src_top + 1 ); 
        TIFF_UINT8_T* src_buf = new TIFF_UINT8_T[ src_buf_size ];
        memset( src_buf , 0 , src_buf_size );

        fseek( _tiff_file->pfile , height_start[src_top] , SEEK_SET );
        fread( src_buf ,src_buf_size , 1 , _tiff_file->pfile );

        TIFF_UINT8_T* buf = new TIFF_UINT8_T[ _tiff_file->samples_per_pixel * _new_tiff_width ];
        memset( buf , 0 , _tiff_file->samples_per_pixel * _new_tiff_width );
        TIFF_UINT8_T* temp_buf = buf ;

        for ( int i_width = 0 ; i_width < _new_tiff_width ; i_width++ )
        {
            double new_y = _geo_rect_des.top;
            new_y -= ( i_height * _scaleY_des ); 
            double new_x = _geo_rect_des.left;
            new_x += ( i_width * _scaleX_des );
#if 1
            if ( _coord_type == 0 )
            {
                _coord_trans.xy_2_latlong( new_x , new_y );
            }
            else
            {
                _coord_trans.xy_2_xy( new_x , new_y );
            }

            new_x = src_geo_xy( new_x );//添加带号

            int pixel_x = 0 , pixel_y = 0 ;
            point_to_by_geo( new_x , new_y , pixel_x , pixel_y );

            if ( !( pixel_y == -1 || pixel_x == -1 ) )
            {   
                memcpy ( temp_buf 
                    , src_buf + ( pixel_y - src_top ) * _tiff_file->samples_per_pixel * _tiff_file->tif_width + pixel_x * _tiff_file->samples_per_pixel 
                    , _tiff_file->samples_per_pixel );

                temp_buf += _tiff_file->samples_per_pixel ;
            }
            else
            {
                temp_buf += _tiff_file->samples_per_pixel ;
            }
#else
            int pixel_x = 0 , pixel_y = 0 ;

            if ( !( pixel_y == -1 || pixel_x == -1 ) )
            {   
                temp_buf += _tiff_file->samples_per_pixel ;
            }
            else
            {
                temp_buf += _tiff_file->samples_per_pixel ;
            }
#endif

        }
        fwrite( buf , _tiff_file->samples_per_pixel * _new_tiff_width , 1 , _line_tiff );
        delete[] buf;
        buf = NULL;

        delete[] src_buf;
        src_buf = NULL ;
    }

    printf( "转换完成！ \n" );
}

void tiffDeform::write_img_data_th_ex()
{
    TIFF_UINT32_T* height_start = new TIFF_UINT32_T[ _new_tiff_height *  sizeof(TIFF_UINT32_T) ];
    memset( height_start , 0 ,  _new_tiff_height *  sizeof(TIFF_UINT32_T) );
    int bits =  _tiff_file->bit_per_samples ;
    if ( bits >= 24 )
    {
        if ( !_tiff_file->tile.is_tile )//不是瓦片数据
        {
            fresh_line_start( _tiff_file , height_start );
        }
    }

    fseek( _line_tiff , 0 , SEEK_END );

    printf( "转换中请稍后... \n" );

    TIFF_UINT8_T* buf = new TIFF_UINT8_T[ _tiff_file->samples_per_pixel * _new_tiff_width ];
    memset( buf , 0 , _tiff_file->samples_per_pixel * _new_tiff_width );
    _coord_trans.init_proj( coord_google , _proj4_str.c_str() );

    geoRECT new_line_rect ;
    new_line_rect.left = _geo_rect_des.left ;
    new_line_rect.right = _geo_rect_des.right ;

    TIFF_UINT32_T LineBytes = _tiff_file->samples_per_pixel * _tiff_file->tif_width;

    FILE* pf = fopen( "1.txt" , "w" );
    char* pwrite = new char[128];
    int first_x = 0 , first_y = 0;
    for ( int i_height = 0 ; i_height < _new_tiff_height ; i_height++ )
    {
        double line_y = _geo_rect_des.top;
        line_y -= ( i_height * _scaleY_des ); 

        new_line_rect.top = line_y ;
        new_line_rect.botton = line_y ;

        geoRECT src_range;
        des_geo_to_src_range( new_line_rect , src_range );

        int src_top = ( int )src_range.top;
        int src_bottom = ( int )src_range.botton;

        TIFF_UINT32_T src_buf_size =  LineBytes * ( src_bottom - src_top + 1 ); 
        TIFF_UINT8_T* src_buf = new TIFF_UINT8_T[ src_buf_size ];
        memset( src_buf , 0 , src_buf_size );

        fseek( _tiff_file->pfile , height_start[src_top] , SEEK_SET );
        fread( src_buf ,src_buf_size , 1 , _tiff_file->pfile );


        TIFF_UINT8_T* temp_buf = buf ;


        memset( pwrite , 0 , 128 );
        sprintf( pwrite , "第 %d 行\n" , i_height );
        fwrite( pwrite , strlen(pwrite) , 1 , pf );

        for ( int i_width = 0 ; i_width < _new_tiff_width ; i_width++ )
        {
            double new_y = _geo_rect_des.top;
            new_y -= ( i_height * _scaleY_des ); 
            double new_x = _geo_rect_des.left;
            new_x += ( i_width * _scaleX_des );

            if ( _coord_type == 0 )
            {
                _coord_trans.xy_2_latlong( new_x , new_y );
            }
            else
            {
                _coord_trans.xy_2_xy( new_x , new_y );
            }

            new_x = src_geo_xy( new_x );//添加带号

            int pixel_x = 0 , pixel_y = 0 ;
            point_to_by_geo( new_x , new_y , pixel_x , pixel_y );

            if ( i_width == 0 )
            {
                first_x = pixel_x;
                first_y = pixel_y;
            }
            memset( pwrite , 0 , 128 );
            sprintf( pwrite , "(%d,%d)" , pixel_x - first_x , pixel_y - first_y );
            fwrite( pwrite , strlen(pwrite) , 1 , pf );

            if ( !( pixel_y == -1 || pixel_x == -1 ) )
            {   
                memcpy ( temp_buf 
                    , src_buf + ( pixel_y - src_top ) * LineBytes + pixel_x * _tiff_file->samples_per_pixel 
                    , _tiff_file->samples_per_pixel );

                temp_buf += _tiff_file->samples_per_pixel ;
            }
            else
            {
                temp_buf += _tiff_file->samples_per_pixel ;
            }
        }
        fwrite( buf , _tiff_file->samples_per_pixel * _new_tiff_width , 1 , _line_tiff );

        fwrite( "\n" , 1 , 1 , pf );

        delete[] src_buf;
        src_buf = NULL ;
    }

    delete[] buf;
    buf = NULL;

    printf( "转换完成！ \n" );
}

void tiffDeform::write_img_data_mul_th()
{
    _height_start = new TIFF_UINT32_T[ _new_tiff_height *  sizeof(TIFF_UINT32_T) ];
    memset( _height_start , 0 ,  _new_tiff_height *  sizeof(TIFF_UINT32_T) );
    int bits =  _tiff_file->bit_per_samples ;
    if ( bits >= 24 )
    {
        if ( !_tiff_file->tile.is_tile )//不是瓦片数据
        {
            fresh_line_start( _tiff_file , _height_start );
        }
    }

    fseek( _line_tiff , 0 , SEEK_END );
    _file_size = ftell( _line_tiff );

    printf( "转换中请稍后... \n" );

    TIFF_UINT8_T* buf = new TIFF_UINT8_T[ _tiff_file->samples_per_pixel * _new_tiff_width ];
    memset( buf , 0 , _tiff_file->samples_per_pixel * _new_tiff_width );
    _coord_trans.init_proj( coord_google , _proj4_str.c_str() );

    geoRECT new_line_rect ;
    new_line_rect.left = _geo_rect_des.left ;
    new_line_rect.right = _geo_rect_des.right ;

    TIFF_UINT32_T LineBytes = _tiff_file->samples_per_pixel * _tiff_file->tif_width;
    TIFF_UINT32_T new_line_bytes = _tiff_file->samples_per_pixel * _new_tiff_width;

    _hMutex = CreateMutex( NULL,FALSE,L"MyFileMutex" );
    m_htiff_write = CreateThread( NULL , 0 , tiff_write_thread , this , 0 , NULL );//启动新线程

    for ( int i_height = 0 ; i_height < _new_tiff_height / 2  ; i_height++ )
    {
        double line_y = _geo_rect_des.top;
        line_y -= ( i_height * _scaleY_des ); 

        new_line_rect.top = line_y ;
        new_line_rect.botton = line_y ;

        geoRECT src_range;
        des_geo_to_src_range( new_line_rect , src_range );

        int src_top = ( int )src_range.top;
        int src_bottom = ( int )src_range.botton;

        TIFF_UINT32_T src_buf_size =  LineBytes * ( src_bottom - src_top + 1 ); 
        TIFF_UINT8_T* src_buf = new TIFF_UINT8_T[ src_buf_size ];
        memset( src_buf , 0 , src_buf_size );

        fseek( _tiff_file->pfile , _height_start[src_top] , SEEK_SET );
        fread( src_buf ,src_buf_size , 1 , _tiff_file->pfile );


        TIFF_UINT8_T* temp_buf = buf ;

        for ( int i_width = 0 ; i_width < _new_tiff_width ; i_width++ )
        {
            double new_y = _geo_rect_des.top;
            new_y -= ( i_height * _scaleY_des ); 
            double new_x = _geo_rect_des.left;
            new_x += ( i_width * _scaleX_des );

            if ( _coord_type == 0 )
            {
                _coord_trans.xy_2_latlong( new_x , new_y );
            }
            else
            {
                _coord_trans.xy_2_xy( new_x , new_y );
            }

            new_x = src_geo_xy( new_x );//添加带号

            int pixel_x = 0 , pixel_y = 0 ;
            point_to_by_geo( new_x , new_y , pixel_x , pixel_y );

            if ( !( pixel_y == -1 || pixel_x == -1 ) )
            {   
                memcpy ( temp_buf 
                    , src_buf + ( pixel_y - src_top ) * LineBytes + pixel_x * _tiff_file->samples_per_pixel 
                    , _tiff_file->samples_per_pixel );

                temp_buf += _tiff_file->samples_per_pixel ;
            }
            else
            {
                temp_buf += _tiff_file->samples_per_pixel ;
            }
        }

        TIFF_UINT64_T temp_file_size = _file_size ;
        temp_file_size += /*_tiff_file->samples_per_pixel * _new_tiff_width*/new_line_bytes * i_height ; 


        WaitForSingleObject( _hMutex ,INFINITE );// //等待临界资源，即锁定文件
        TIFF_UINT64_T ret = /*fseek*/_fseeki64( _line_tiff , temp_file_size , SEEK_SET );
        int write_ret = fwrite( buf , /*_tiff_file->samples_per_pixel * _new_tiff_width*/new_line_bytes , 1 , _line_tiff );
        ReleaseMutex( _hMutex );//释放互斥量，解除锁定

        delete[] src_buf;
        src_buf = NULL ;
    }

    delete[] buf;
    buf = NULL;

    printf( "转换完成！ \n" );
}

DWORD WINAPI tiffDeform::tiff_write_thread( LPVOID lpParameter ) 
{
    tiffDeform* pApp=( tiffDeform* )lpParameter;


    printf( "转换中请稍后... \n" );

    TIFF_UINT8_T* buf = new TIFF_UINT8_T[ pApp->_tiff_file->samples_per_pixel * pApp->_new_tiff_width ];
    memset( buf , 0 , pApp->_tiff_file->samples_per_pixel * pApp->_new_tiff_width );
    //pApp->_coord_trans.init_proj( coord_google , pApp->_proj4_str.c_str() );
    geoRECT new_line_rect ;
    new_line_rect.left = pApp->_geo_rect_des.left ;
    new_line_rect.right = pApp->_geo_rect_des.right ;

    TIFF_UINT32_T LineBytes = pApp->_tiff_file->samples_per_pixel * pApp->_tiff_file->tif_width ;
    TIFF_UINT32_T new_line_bytes = pApp->_tiff_file->samples_per_pixel * pApp->_new_tiff_width;

    for ( int i_height = pApp->_new_tiff_height / 2 ; i_height < pApp->_new_tiff_height  ; i_height++ )
    {
        double line_y = pApp->_geo_rect_des.top;
        line_y -= ( i_height * pApp->_scaleY_des ); 

        new_line_rect.top = line_y ;
        new_line_rect.botton = line_y ;

        geoRECT src_range;
        pApp->des_geo_to_src_range( new_line_rect , src_range );

        int src_top = ( int )src_range.top;
        int src_bottom = ( int )src_range.botton;

        TIFF_UINT32_T src_buf_size =  LineBytes * ( src_bottom - src_top + 1 ); 
        TIFF_UINT8_T* src_buf = new TIFF_UINT8_T[ src_buf_size ];
        memset( src_buf , 0 , src_buf_size );

        fseek( pApp->_tiff_file->pfile , pApp->_height_start[src_top] , SEEK_SET );
        fread( src_buf ,src_buf_size , 1 , pApp->_tiff_file->pfile );


        TIFF_UINT8_T* temp_buf = buf ;

        for ( int i_width = 0 ; i_width < pApp->_new_tiff_width ; i_width++ )
        {
            double new_y = pApp->_geo_rect_des.top;
            new_y -= ( i_height * pApp->_scaleY_des ); 
            double new_x = pApp->_geo_rect_des.left;
            new_x += ( i_width * pApp->_scaleX_des );

            if ( pApp->_coord_type == 0 )
            {
                pApp->_coord_trans.xy_2_latlong( new_x , new_y );
            }
            else
            {
                pApp->_coord_trans.xy_2_xy( new_x , new_y );
            }

            new_x = pApp->src_geo_xy( new_x );//添加带号

            int pixel_x = 0 , pixel_y = 0 ;
            pApp->point_to_by_geo( new_x , new_y , pixel_x , pixel_y );

            if ( !( pixel_y == -1 || pixel_x == -1 ) )
            {   
                memcpy ( temp_buf 
                    , src_buf + ( pixel_y - src_top ) * LineBytes + pixel_x * pApp->_tiff_file->samples_per_pixel 
                    , pApp->_tiff_file->samples_per_pixel );

                temp_buf += pApp->_tiff_file->samples_per_pixel ;
            }
            else
            {
                temp_buf += pApp->_tiff_file->samples_per_pixel ;
            }
        }

        TIFF_UINT64_T temp_file_size = pApp->_file_size ;
        temp_file_size += /*pApp->_tiff_file->samples_per_pixel * pApp->_new_tiff_width*/new_line_bytes * i_height ; 

        WaitForSingleObject( pApp->_hMutex ,INFINITE );// //等待临界资源，即锁定文件
        TIFF_UINT64_T ret = /*fseek*/_fseeki64( pApp->_line_tiff , temp_file_size , SEEK_SET );
        int write_ret = fwrite( buf , /*pApp->_tiff_file->samples_per_pixel * pApp->_new_tiff_width*/ new_line_bytes , 1 , pApp->_line_tiff );
        ReleaseMutex( pApp->_hMutex );//释放互斥量，解除锁定

        delete[] src_buf;
        src_buf = NULL ;
    }

    delete[] buf;
    buf = NULL;

    return 0;
}

void tiffDeform::write_img_by_record()
{
    TIFF_UINT32_T* height_start = new TIFF_UINT32_T[ _new_tiff_height *  sizeof(TIFF_UINT32_T) ];
    memset( height_start , 0 ,  _new_tiff_height *  sizeof(TIFF_UINT32_T) );
    int bits =  _tiff_file->bit_per_samples ;
    if ( bits >= 8 )
    {
        if ( !_tiff_file->tile.is_tile )//不是瓦片数据
        {
            fresh_line_start( _tiff_file , height_start );
        }
    }

    _fseeki64( _line_tiff , 0 , SEEK_END );

    printf( "转换中请稍后... \n" );

    TIFF_UINT8_T* buf = new TIFF_UINT8_T[ _tiff_file->samples_per_pixel * _new_tiff_width ];
    memset( buf , 0 , _tiff_file->samples_per_pixel * _new_tiff_width );
    _coord_trans.init_proj( coord_google , _proj4_str.c_str() );

    geoRECT new_line_rect ;
    new_line_rect.left = _geo_rect_des.left ;
    new_line_rect.right = _geo_rect_des.right ;

    TIFF_UINT32_T LineBytes = _tiff_file->samples_per_pixel * _tiff_file->tif_width;

    //计算中间行的斜率
    double line_y = _geo_rect_des.top;
    line_y -= ( _tiff_file->tif_height / 2 * _scaleY_des ); 
    new_line_rect.top = line_y ;
    new_line_rect.botton = line_y ;

    geoRECT src_range;
    des_geo_to_src_range( new_line_rect , src_range );
    int src_top = ( int )src_range.top;
    int src_bottom = ( int )src_range.botton;

    TIFF_UINT32_T src_buf_size =  LineBytes * ( src_bottom - src_top + 1 ); 
    TIFF_UINT8_T* src_buf = new TIFF_UINT8_T[ src_buf_size ];
    memset( src_buf , 0 , src_buf_size );

    _fseeki64( _tiff_file->pfile , height_start[src_top] , SEEK_SET );
    fread( src_buf ,src_buf_size , 1 , _tiff_file->pfile );

    int *diff_x = new int[ _new_tiff_width ] ;memset( diff_x , 0 , _new_tiff_width );
    int *diff_y = new int[ _new_tiff_width ] ;memset( diff_y , 0 , _new_tiff_width );
    int first_x = 0 , first_y = 0;
    for ( int i_width = 0 ; i_width < _new_tiff_width ; i_width++ )
    {
        double new_y = new_line_rect.top;
        double new_x = _geo_rect_des.left;
        new_x += ( i_width * _scaleX_des );

        if ( _coord_type == 0 )
        {
            _coord_trans.xy_2_latlong( new_x , new_y );
        }
        else
        {
            _coord_trans.xy_2_xy( new_x , new_y );
        }

        new_x = src_geo_xy( new_x );//添加带号

        int pixel_x = 0 , pixel_y = 0 ;
        point_to_by_geo_ex( new_x , new_y , pixel_x , pixel_y );

        if ( i_width == 0 )
        {
            diff_x[0] = 0 ;
            diff_y[0] = 0 ;
            first_x = pixel_x ;
            first_y = pixel_y ;
        }
        else
        {
            diff_x[ i_width ] = pixel_x - first_x;
            diff_y[ i_width ] = pixel_y - first_y;
        }
    }

    for ( int i_height = 0 ; i_height < _new_tiff_height ; i_height++ )
    {

        int pixel_first_x = 0 , pixel_first_y = 0 ;

        double line_y = _geo_rect_des.top;
        line_y -= ( i_height * _scaleY_des ); 

        new_line_rect.top = line_y ;
        new_line_rect.botton = line_y ;

        geoRECT src_range;
        des_geo_to_src_range_first( new_line_rect , src_range , pixel_first_x , pixel_first_y );

        int src_top = ( int )src_range.top;
        int src_bottom = ( int )src_range.botton;

        TIFF_UINT32_T src_buf_size =  LineBytes * ( src_bottom - src_top + 1 ); 
        TIFF_UINT8_T* src_buf = new TIFF_UINT8_T[ src_buf_size ];
        memset( src_buf , 0 , src_buf_size );

        _fseeki64( _tiff_file->pfile , height_start[src_top] , SEEK_SET );
        fread( src_buf ,src_buf_size , 1 , _tiff_file->pfile );

        TIFF_UINT8_T* temp_buf = buf ;

        for ( int i_width = 0 ; i_width < _new_tiff_width ; i_width++ )
        {
            int pixel_x = 0 , pixel_y = 0 ;

            pixel_x = /*( int )src_range.left*/pixel_first_x + diff_x[ i_width ];
            pixel_y = /*( int )src_range.top*/pixel_first_y + diff_y[ i_width ];

            if ( pixel_y <  (int)src_range.top || pixel_y > (int) src_range.botton )
            {
                pixel_y = -1;
            }
            if ( pixel_x < 0 || pixel_x > _tiff_file->tif_width )
            {
                pixel_x = 0;
            }
            if ( !( pixel_y == -1 || pixel_x == -1 ) )
            {   
                memcpy ( temp_buf 
                    , src_buf + ( pixel_y - src_top ) * LineBytes + pixel_x * _tiff_file->samples_per_pixel 
                    , _tiff_file->samples_per_pixel );

                temp_buf += _tiff_file->samples_per_pixel ;
            }
            else
            {
                temp_buf += _tiff_file->samples_per_pixel ;
            }
        }
        fwrite( buf , _tiff_file->samples_per_pixel * _new_tiff_width , 1 , _line_tiff );

        delete[] src_buf;
        src_buf = NULL ;
    }

    delete[] buf;
    buf = NULL;

    //printf( "转换完成！ \n" );
}

void tiffDeform::write_img_by_block_record( )
{
    TIFF_UINT32_T* height_start = new TIFF_UINT32_T[ _new_tiff_height *  sizeof( TIFF_UINT32_T ) ];
    memset( height_start , 0 ,  _new_tiff_height *  sizeof(TIFF_UINT32_T) );
    int bits =  _tiff_file->bit_per_samples ;
    if ( bits >= 24 )
    {
        if ( !_tiff_file->tile.is_tile )//不是瓦片数据
        {
            fresh_line_start( _tiff_file , height_start );
        }
    }

    fseek( _line_tiff , 0 , SEEK_END );

    int bc = 0;
    int blockCount = 64;
    DiffValue** diff = (DiffValue **)malloc(blockCount * sizeof(DiffValue *));;
    for(bc=0; bc<blockCount; bc++){
        diff[bc] = new DiffValue[ _new_tiff_width ];
        memset( diff[bc] , 0 , sizeof( DiffValue ) );
    }

    //分成四部分分别计算差值
/*  DiffValue* diff1 = new DiffValue[ _new_tiff_width ];
    memset( diff1 , 0 , sizeof( DiffValue ) );
    DiffValue* diff2 = new DiffValue[ _new_tiff_width ];
    memset( diff2 , 0 , sizeof( DiffValue ) );
    DiffValue* diff3 = new DiffValue[ _new_tiff_width ];
    memset( diff3 , 0 , sizeof( DiffValue ) );
    DiffValue* diff4 = new DiffValue[ _new_tiff_width ];
    memset( diff4 , 0 , sizeof( DiffValue ) );*/

    int block_interval = _new_tiff_height / blockCount ;

/*  TIFF_UINT16_T height_pos[4] = { 0 , 0 , 0 , 0 } ;         //每部分的中间位置
    height_pos[0] = block_interval / 2;                       //第一部分的中间行
    height_pos[1] = block_interval + block_interval / 2;      //第二部分的中间位置
    height_pos[2] = _new_tiff_height / 2 + block_interval / 2;//第三部分的中间位置行
    height_pos[3] = _new_tiff_height - block_interval / 2;    //第四部分的中间位置行*/

    TIFF_UINT16_T *height_pos = (TIFF_UINT16_T *)malloc(sizeof(TIFF_UINT16_T) * blockCount);
    for(bc=0; bc<blockCount; bc++)
    {
        height_pos[bc] = block_interval / 2 + block_interval * bc;                       //第一部分的中间行
    }
    _coord_trans.init_proj( coord_google , _proj4_str.c_str() );//初始化proj4

/*  geoRECT temp_geo2 , temp_geo1 , temp_geo3 , temp_geo4 ;
    memset( &temp_geo1 , 0 , sizeof( geoRECT ) );
    memset( &temp_geo2 , 0 , sizeof( geoRECT ) );
    memset( &temp_geo3 , 0 , sizeof( geoRECT ) );
    memset( &temp_geo4 , 0 , sizeof( geoRECT ) );

    temp_geo1.left = _geo_rect_des.left ;
    temp_geo1.top = _geo_rect_des.top ;
    temp_geo1.top -= ( height_pos[0] * _scaleY_des );

    temp_geo2.left = _geo_rect_des.left ;
    temp_geo2.top = _geo_rect_des.top ;
    temp_geo2.top -= ( height_pos[1] * _scaleY_des ); 

    temp_geo3.left = _geo_rect_des.left ;
    temp_geo3.top = _geo_rect_des.top ;
    temp_geo3.top -= ( height_pos[2] * _scaleY_des ); 

    temp_geo4.left = _geo_rect_des.left ;
    temp_geo4.top = _geo_rect_des.top ;
    temp_geo4.top -= ( height_pos[3] * _scaleY_des ); 

    int first1_x = 0 , first1_y = 0 ;
    int first2_x = 0 , first2_y = 0 ;
    int first3_x = 0 , first3_y = 0 ;
    int first4_x = 0 , first4_y = 0 ;*/

    geoRECT *temp_geo = (geoRECT *)malloc(sizeof(geoRECT) * blockCount);
    int *first_x = (int *)malloc(sizeof(int) * blockCount);
    int *first_y = (int *)malloc(sizeof(int) * blockCount);
    for(bc=0; bc<blockCount; bc++)
    {
        memset( &temp_geo[bc] , 0 , sizeof( geoRECT ) );

        temp_geo[bc].left = _geo_rect_des.left ;
        temp_geo[bc].top = _geo_rect_des.top ;
        temp_geo[bc].top -= ( height_pos[bc] * _scaleY_des );

        first_x[bc] = 0;
        first_y[bc] = 0;
    }

    TIFF_UINT32_T LineBytes = _tiff_file->samples_per_pixel * _tiff_file->tif_width;

    for ( int i_new_w = 0 ; i_new_w < _new_tiff_width ; i_new_w ++ )
    {
        for( bc = 0 ; bc < blockCount ; bc ++ )
        {
            double new_y = temp_geo[bc].top;
            double new_x = _geo_rect_des.left;
            new_x += ( i_new_w * _scaleX_des );

            if ( _coord_type == 0 )
            {
                _coord_trans.xy_2_latlong( new_x , new_y );
            }
            else
            {
                _coord_trans.xy_2_xy( new_x , new_y );
            }

            new_x = src_geo_xy( new_x );//添加带号

            int pixel_x = 0 , pixel_y = 0 ;
            point_to_by_geo( new_x , new_y , pixel_x , pixel_y );

            if ( i_new_w == 0 )
            {
                double new_y = temp_geo[bc].top;
                double new_x = _geo_rect_des.left;
                new_x += ( _new_tiff_width /2 * _scaleX_des );
                if ( _coord_type == 0 )
                {
                    _coord_trans.xy_2_latlong( new_x , new_y );
                }
                else
                {
                    _coord_trans.xy_2_xy( new_x , new_y );
                }

                new_x = src_geo_xy( new_x );//添加带号

                int pixel_x = 0 , pixel_y = 0 ;
                point_to_by_geo( new_x , new_y , pixel_x , pixel_y );

                diff[bc][_new_tiff_width /2].diff_x = 0 ;
                diff[bc][_new_tiff_width /2].diff_y = 0 ;
                first_x[bc] = pixel_x ;
                first_y[bc] = pixel_y ;
            }
            else
            {
                diff[bc][ i_new_w ].diff_x = pixel_x - first_x[bc] ;
                diff[bc][ i_new_w ].diff_y = pixel_y - first_y[bc] ;
            }
        }
/*      //第一部分
        double new_y = temp_geo1.top;
        double new_x = _geo_rect_des.left;
        new_x += ( i_new_w * _scaleX_des );

        if ( _coord_type == 0 )
        {
            _coord_trans.xy_2_latlong( new_x , new_y );
        }
        else
        {
            _coord_trans.xy_2_xy( new_x , new_y );
        }

        new_x = src_geo_xy( new_x );//添加带号

        int pixel_x = 0 , pixel_y = 0 ;
        point_to_by_geo( new_x , new_y , pixel_x , pixel_y );

        if ( i_new_w == 0 )
        {
            diff1[0].diff_x = 0 ;
            diff1[0].diff_y = 0 ;
            first1_x = pixel_x ;
            first1_y = pixel_y ;
        }
        else
        {
            diff1[ i_new_w ].diff_x = pixel_x - first1_x ;
            diff1[ i_new_w ].diff_y = pixel_y - first1_y ;
        }

        //第二部分
        new_y = temp_geo2.top;
        new_x = _geo_rect_des.left;
        new_x += ( i_new_w * _scaleX_des );

        if ( _coord_type == 0 )
        {
            _coord_trans.xy_2_latlong( new_x , new_y );
        }
        else
        {
            _coord_trans.xy_2_xy( new_x , new_y );
        }

        new_x = src_geo_xy( new_x );//添加带号

        pixel_x = 0 , pixel_y = 0 ;
        point_to_by_geo( new_x , new_y , pixel_x , pixel_y );

        if ( i_new_w == 0 )
        {
            diff2[0].diff_x = 0 ;
            diff2[0].diff_y = 0 ;
            first2_x = pixel_x ;
            first2_y = pixel_y ;
        }
        else
        {
            diff2[ i_new_w ].diff_x = pixel_x - first2_x ;
            diff2[ i_new_w ].diff_y = pixel_y - first2_y ;
        }

        //第三部分
        new_y = temp_geo3.top;
        new_x = _geo_rect_des.left;
        new_x += ( i_new_w * _scaleX_des );

        if ( _coord_type == 0 )
        {
            _coord_trans.xy_2_latlong( new_x , new_y );
        }
        else
        {
            _coord_trans.xy_2_xy( new_x , new_y );
        }

        new_x = src_geo_xy( new_x );//添加带号

        pixel_x = 0 , pixel_y = 0 ;
        point_to_by_geo( new_x , new_y , pixel_x , pixel_y );

        if ( i_new_w == 0 )
        {
            diff3[0].diff_x = 0 ;
            diff3[0].diff_y = 0 ;
            first3_x = pixel_x ;
            first3_y = pixel_y ;
        }
        else
        {
            diff3[ i_new_w ].diff_x = pixel_x - first3_x ;
            diff3[ i_new_w ].diff_y = pixel_y - first3_y ;
        }

        //第四部分
        new_y = temp_geo4.top;
        new_x = _geo_rect_des.left;
        new_x += ( i_new_w * _scaleX_des );

        if ( _coord_type == 0 )
        {
            _coord_trans.xy_2_latlong( new_x , new_y );
        }
        else
        {
            _coord_trans.xy_2_xy( new_x , new_y );
        }

        new_x = src_geo_xy( new_x );//添加带号

        pixel_x = 0 , pixel_y = 0 ;
        point_to_by_geo( new_x , new_y , pixel_x , pixel_y );

        if ( i_new_w == 0 )
        {
            diff4[0].diff_x = 0 ;
            diff4[0].diff_y = 0 ;
            first4_x = pixel_x ;
            first4_y = pixel_y ;
        }
        else
        {
            diff4[ i_new_w ].diff_x = pixel_x - first4_x ;
            diff4[ i_new_w ].diff_y = pixel_y - first4_y ;
        }*/
    }

    geoRECT new_line_rect ;
    memset( &new_line_rect , 0 , sizeof( geoRECT ) );
    new_line_rect.left = _geo_rect_des.left ;
    new_line_rect.right = _geo_rect_des.right ;

    TIFF_UINT8_T* buf = new TIFF_UINT8_T[ _tiff_file->samples_per_pixel * _new_tiff_width ];
    memset( buf , 0 , _tiff_file->samples_per_pixel * _new_tiff_width );

    for ( int i_height = 0 ; i_height < _new_tiff_height ; i_height++ )
    {
        double line_y = _geo_rect_des.top;
        line_y -= ( i_height * _scaleY_des ); 

        new_line_rect.top = line_y ;
        new_line_rect.botton = line_y ;

        geoRECT src_range;
        des_geo_to_src_range( new_line_rect , src_range );

        int src_top = ( int )src_range.top;
        int src_bottom = ( int )src_range.botton;

        TIFF_UINT32_T src_buf_size =  LineBytes * ( src_bottom - src_top + 1 ); 
        TIFF_UINT8_T* src_buf = new TIFF_UINT8_T[ src_buf_size ];
        memset( src_buf , 0 , src_buf_size );

        fseek( _tiff_file->pfile , height_start[src_top] , SEEK_SET );
        fread( src_buf ,src_buf_size , 1 , _tiff_file->pfile );

        TIFF_UINT8_T* temp_buf = buf ;

        DiffValue* ptemp = NULL ;

        for(bc=0; bc<blockCount; bc++)
        {
            if(i_height <= block_interval * (bc+1) && i_height >= block_interval * (bc))
            {
                ptemp = diff[bc];
            }
        }
        if(!ptemp) break;
/*      if ( i_height <= block_interval )
        {
            ptemp = diff1;
        }
        else if ( i_height > block_interval && i_height <= _new_tiff_height/2 )
        {
            ptemp = diff2;
        }
        else if ( i_height > _new_tiff_height/2 && i_height < ( _new_tiff_height/2 + block_interval ) )
        {
            ptemp = diff3 ;
        }
        else if ( i_height >= ( _new_tiff_height/2 + block_interval ) )
        {
            ptemp = diff4;
        }*/

        for ( int i_width = 0 ; i_width < _new_tiff_width ; i_width++ )
        {
            int pixel_x = 0 , pixel_y = 0 ;
            //if( i_width <= _new_tiff_width/2 )
            /*{
                pixel_x = ( int )src_range.left - ptemp[i_width].diff_x ;
                pixel_y = ( int )src_range.botton - ptemp[i_width].diff_y ;
            }*/
            /*else*/
            {
                pixel_x = ( int )src_range.left + ptemp[i_width].diff_x ;
                pixel_y = ( int )src_range.botton + ptemp[i_width].diff_y ;
            }

            if ( pixel_y <  (int)src_range.top || pixel_y > (int) src_range.botton )
            {
                pixel_y = -1;
            }
            if ( pixel_x < 0 || pixel_x > _tiff_file->tif_width )
            {
                pixel_x = 0;
            }
            if ( !( pixel_y == -1 || pixel_x == -1 ) )
            {   
                memcpy ( temp_buf 
                    , src_buf + ( pixel_y - src_top ) * LineBytes + pixel_x * _tiff_file->samples_per_pixel 
                    , _tiff_file->samples_per_pixel );

                temp_buf += _tiff_file->samples_per_pixel ;
            }
            else
            {
                temp_buf += _tiff_file->samples_per_pixel ;
            }
        }
        fwrite( buf , _tiff_file->samples_per_pixel * _new_tiff_width , 1 , _line_tiff );

        delete[] src_buf;
        src_buf = NULL ;
    }

/*  delete[] diff1;
    diff1 = NULL ;
    delete[] diff2;
    diff2 = NULL ;
    delete[] diff3;
    diff3 = NULL ;
    delete[] diff4;
    diff4 = NULL ;*/
}

void tiffDeform::write_img_by_sin()
{
    TIFF_UINT32_T* height_start = new TIFF_UINT32_T[ _new_tiff_height *  sizeof(TIFF_UINT32_T) ];
    memset( height_start , 0 ,  _new_tiff_height *  sizeof(TIFF_UINT32_T) );
    int bits =  _tiff_file->bit_per_samples ;
    if ( bits >= 24 )
    {
        if ( !_tiff_file->tile.is_tile )//不是瓦片数据
        {
            fresh_line_start( _tiff_file , height_start );
        }
    }

    fseek( _line_tiff , 0 , SEEK_END );

    printf( "转换中请稍后... \n" );

    TIFF_UINT8_T* buf = new TIFF_UINT8_T[ _tiff_file->samples_per_pixel * _new_tiff_width ];
    memset( buf , 0 , _tiff_file->samples_per_pixel * _new_tiff_width );
    _coord_trans.init_proj( coord_google , _proj4_str.c_str() );

    geoRECT new_line_rect ;
    new_line_rect.left = _geo_rect_des.left ;
    new_line_rect.right = _geo_rect_des.right ;

    TIFF_UINT32_T LineBytes = _tiff_file->samples_per_pixel * _tiff_file->tif_width;

    //计算中间行的斜率
    double line_y = _geo_rect_des.top;
    line_y -= ( _tiff_file->tif_height / 2 * _scaleY_des ); 
    new_line_rect.top = line_y ;
    new_line_rect.botton = line_y ;

    geoRECT new_line_rect_2 = new_line_rect ;
    to_src_proj( new_line_rect_2 , /*true*/false );
    //倾斜角的正切值
    double slope_tan = ( new_line_rect_2.top - new_line_rect_2.botton ) / ( new_line_rect_2.right - new_line_rect_2.left );

    //倾斜角的正弦值
    double hypotenuse_line = sqrt( ( new_line_rect_2.top - new_line_rect_2.botton )*( new_line_rect_2.top - new_line_rect_2.botton ) 
        + ( new_line_rect_2.right - new_line_rect_2.left )*( new_line_rect_2.right - new_line_rect_2.left ) );
    double slope_sin = ( new_line_rect_2.top - new_line_rect_2.botton ) / hypotenuse_line ;
    //倾斜角的余弦值
    double slope_cos = ( new_line_rect_2.right - new_line_rect_2.left ) / hypotenuse_line;
    //X方向上的放大比例
    double scale_x = ( double )_tiff_file->tif_width / _new_tiff_width ;

    geoRECT src_range;
    des_geo_to_src_range( new_line_rect , src_range );
    int src_top = ( int )src_range.top;
    int src_bottom = ( int )src_range.botton;

    TIFF_UINT32_T src_buf_size =  LineBytes * ( src_bottom - src_top + 1 ); 
    TIFF_UINT8_T* src_buf = new TIFF_UINT8_T[ src_buf_size ];
    memset( src_buf , 0 , src_buf_size );

    fseek( _tiff_file->pfile , height_start[src_top] , SEEK_SET );
    fread( src_buf ,src_buf_size , 1 , _tiff_file->pfile );

    for ( int i_height = 0 ; i_height < _new_tiff_height ; i_height++ )
    {
        double line_y = _geo_rect_des.top;
        line_y -= ( i_height * _scaleY_des ); 

        new_line_rect.top = line_y ;
        new_line_rect.botton = line_y ;

        geoRECT src_range;
        des_geo_to_src_range( new_line_rect , src_range );

        int src_top = ( int )src_range.top;
        int src_bottom = ( int )src_range.botton;

        TIFF_UINT32_T src_buf_size =  LineBytes * ( src_bottom - src_top + 1 ); 
        TIFF_UINT8_T* src_buf = new TIFF_UINT8_T[ src_buf_size ];
        memset( src_buf , 0 , src_buf_size );

        fseek( _tiff_file->pfile , height_start[src_top] , SEEK_SET );
        fread( src_buf ,src_buf_size , 1 , _tiff_file->pfile );

        //计算第一个点坐标
        int pixel_first_x = 0 , pixel_first_y = 0 ;
        {
            double new_y = _geo_rect_des.top;
            new_y -= ( i_height * _scaleY_des ); 
            double new_x = _geo_rect_des.left;

            if ( _coord_type == 0 )
            {
                _coord_trans.xy_2_latlong( new_x , new_y );
            }
            else
            {
                _coord_trans.xy_2_xy( new_x , new_y );
            }

            new_x = src_geo_xy( new_x );


            point_to_by_geo_ex( new_x , new_y , pixel_first_x , pixel_first_y );
        }

        TIFF_UINT8_T* temp_buf = buf ;

        for ( int i_width = 0 ; i_width < _new_tiff_width ; i_width ++ )
        {
            int pixel_x = 0 , pixel_y = 0 ;
            pixel_x = /*( int )src_range.left*/( int )( pixel_first_x + ( i_width * scale_x + 0.5 )) ;
            pixel_y = /*( int )src_range.botton*/( int )( pixel_first_y - ( pixel_x * slope_tan + 0.5 )) ;

            if ( pixel_y <  (int)src_range.top || pixel_y > (int) src_range.botton )
            {
                pixel_y = -1;
            }
            if ( pixel_x < 0 || pixel_x > _tiff_file->tif_width )
            {
                pixel_x = 0;
            }
            if ( !( pixel_y == -1 || pixel_x == -1 ) )
            {   
                memcpy ( temp_buf 
                    , src_buf + ( pixel_y - src_top ) * LineBytes + pixel_x * _tiff_file->samples_per_pixel 
                    , _tiff_file->samples_per_pixel );

                temp_buf += _tiff_file->samples_per_pixel ;
            }
            else
            {
                temp_buf += _tiff_file->samples_per_pixel ;
            }
        }
        fwrite( buf , _tiff_file->samples_per_pixel * _new_tiff_width , 1 , _line_tiff );

        delete[] src_buf;
        src_buf = NULL ;
    }

    delete[] buf;
    buf = NULL;

    //printf( "转换完成！ \n" );
}

void tiffDeform::point_to_by_geo( double x , double y , int&pt_x , int&pt_y )
{
    pt_x = (int)(( x - _geo_rect_src.left )/_scaleX_src + 0.5 );//计算像素点的在原始图片的范围
    pt_y = (int)( ( _geo_rect_src.top - y ) / _scaleY_src + 0.5 );

    if ( !(pt_x <= _tiff_file->tif_width && pt_x >= 0) )//像素点出了范围
    {
        pt_x = -1;
    }

    if( !(pt_y <= _tiff_file->tif_height && pt_y >= 0 ) )//像素点出了范围
    {
        pt_y = -1;
    }

}

void tiffDeform::point_to_by_geo_ex( double x , double y , int&pt_x , int&pt_y )
{
    pt_x = (int)(( x - _geo_rect_src.left )/_scaleX_src + 0.5 );//计算像素点的在原始图片的范围
    pt_y = (int)( ( _geo_rect_src.top - y ) / _scaleY_src + 0.5 );
}

string tiffDeform::new_tiff_name()
{
    int pos = _tiff_path.rfind( '\\' );
    string tiff_name = _tiff_path.substr( pos + 1 );
    string dir = _tiff_path.substr( 0 , pos + 1 );
    string temp_name = "line_";
    temp_name += tiff_name ;
    return ( dir + temp_name ) ;
}

void tiffDeform::write_tag_list()
{
    for ( int i = 0 ; i < _de_num ; i++ )
    {
        fseek( _line_tiff , DE_START + ONE_DE_SIZE * i , SEEK_SET );
        fwrite( &( de_list[i].de.tag ) , 2 , 1 , _line_tiff );   //TAG 2字节
        fwrite( &( de_list[i].de.type ) , 2 , 1 , _line_tiff );  //数据类型 2字节
        fwrite( &( de_list[i].de.count ) , 4 , 1 , _line_tiff ); //count 4字节    

        if( de_list[i].de.tag == 273 )//Strip offset
        {
            fseek( _line_tiff , 0 , SEEK_END );
            _strip_offset_pos = ftell( _line_tiff );
        }

        //写入offset
        if( de_list[i].data_source == 0 )//直接写入值
        {
            fwrite( &( de_list[i].de.offset ) , 4 , 1 , _line_tiff );
        }
        else if ( de_list[i].data_source == 1 )//文件对应的偏移量
        {
            fseek( _line_tiff , 0 , SEEK_END );

            TIFF_UINT32_T pos = ftell( _line_tiff );

            TIFF_UINT64_T buffer_size = data_type_list[de_list[i].de.type].type_size * de_list[i].de.count;
            file_disk_data( de_list[i].de , buffer_size );

            //修改TAG对应的数据存放的地址
            fseek( _line_tiff , DE_START + ONE_DE_SIZE * i + 8 , SEEK_SET );
            fwrite( &pos , 1 , 4 , _line_tiff );
        }
        else if ( de_list[i].data_source == 2 )//内存
        {
            fseek( _line_tiff , 0 , SEEK_END );
            TIFF_UINT32_T pos = ftell( _line_tiff );
            fwrite ( de_list[i].mem_data , 1 ,  data_type_list[de_list[i].de.type].type_size * de_list[i].de.count  , _line_tiff );

            //修改TAG对应的数据存放的地址
            fseek( _line_tiff , DE_START + ONE_DE_SIZE * i + 8 , SEEK_SET );
            fwrite( &pos , 1 , 4 , _line_tiff );

            if ( de_list[i].de.tag == 33550 
                || de_list[i].de.tag == 33922
                || de_list[i].de.tag == 34735
                || de_list[i].de.tag == 34736
                || de_list[i].de.tag == 34737)//这些属于静态内存不需要释放
            {
                continue;
            }
            delete[] de_list[i].mem_data;
        }
    }
}

TIFF_UINT64_T tiffDeform::file_disk_data( DirectoryEntry de , TIFF_UINT64_T buffer_size )
{
    fseek( _tiff_file->pfile , de.offset , SEEK_SET );

    TIFF_UINT8_T* buf = new TIFF_UINT8_T[1024];
    memset( buf , 0 , 1024 );

    TIFF_UINT64_T fs = 0;
    TIFF_UINT16_T read_size = 0;
    if ( buffer_size <= 1024 )//若小于1024字节，则读取之后直接写入即可
    {
        read_size = fread( buf , 1 , buffer_size , _tiff_file->pfile );

        if( _tiff_file->tiff_byte_order == TIFF_BIGENDIAN && data_type_list[de.type].type_size != 1 )
        {
            sort_byte_order( buf , de.type , de.count );
        }

        fs += fwrite ( buf , 1 , read_size , _line_tiff );
    }
    else//若大于1024字节，则分批写入1024字节，最后写入不足1024的字节
    {
        TIFF_UINT16_T tile_num = ( int )(buffer_size / 1024) ;
        TIFF_UINT16_T last_num = buffer_size % 1024;
        for ( int i = 0 ; i < tile_num ; i++ )
        {
            read_size = fread( buf , 1 , 1024 , _tiff_file->pfile );//注意参数的顺序
            if( _tiff_file->tiff_byte_order == TIFF_BIGENDIAN && data_type_list[de.type].type_size != 1 )
            {
                sort_byte_order( buf , de.type , de.count );
            }
            fs += fwrite ( buf , 1 , read_size , _line_tiff );
        }

        read_size = fread( buf , 1 , last_num , _tiff_file->pfile );
        if( _tiff_file->tiff_byte_order == TIFF_BIGENDIAN && data_type_list[de.type].type_size != 1 )
        {
            sort_byte_order( buf , de.type , de.count );
        }
        fs += fwrite ( buf , 1 , last_num , _line_tiff );
    }
    delete[] buf;
    buf = NULL;
    return fs;
}

void tiffDeform::sort_byte_order( TIFF_UINT8_T* buf , TIFF_UINT16_T data_type , TIFF_UINT16_T data_count )
{
    TIFF_UINT8_T* p = buf;

    for ( TIFF_UINT16_T i = 0 ; i < data_count ; i++ )
    {
        if ( data_type == 3 || data_type == 8 )//SHORT
        {
            TIFF_UINT16_T ret = sget2( p , TIFF_BIGENDIAN );
            memcpy( p , &ret , 2 );
            p += 2;
        }
        else if ( data_type == 4 ||  data_type == 9 ||  data_type == 11 )//LONG
        {
            TIFF_UINT32_T ret = sget4( p , TIFF_BIGENDIAN );
            memcpy( p , &ret , 4 );
            p += 4;
        }
        if ( data_type == 5 || data_type == 10 )
        {
            TIFF_UINT32_T ret = sget4( p , TIFF_BIGENDIAN );
            memcpy( p , &ret , 4 );
            p += 4;

            ret = sget4( p , TIFF_BIGENDIAN );
            memcpy( p , &ret , 4 );
            p += 4;
        }
        else if ( data_type == 12 )//DOUBLE
        {
            TIFF_UINT64_T ret = sget8( p , TIFF_BIGENDIAN );
            memcpy( p , &ret , 8 );
            p += 8;
        }
    }
}

//修改strip offset所对应的值
void tiffDeform::modify_strip_offset()
{
    fseek( _line_tiff , 0 , SEEK_END );
    TIFF_UINT32_T current_size = ftell( _line_tiff );
    fseek( _line_tiff , _strip_offset_pos , SEEK_SET );

    TIFF_UINT32_T width_bytes = _new_tiff_width * _tiff_file->samples_per_pixel ;
    for ( int i = 0 ; i < _new_tiff_height ; i++ )
    {
        TIFF_UINT32_T height_start = current_size + i * width_bytes;
        fwrite( &height_start , 1 , 4 , _line_tiff );
    }
}

void tiffDeform::write_file_header( )
{
    //字节序   
    fwrite( "II" , 2 , 1 , _line_tiff );

    //版本号
    TIFF_UINT16_T ver = 0x002a;
    fwrite( &ver , 2 , 1 , _line_tiff );

    //Tag偏移量
    TIFF_UINT32_T offset = 0x00000008;
    fwrite( &offset , 4 , 1 , _line_tiff );
}

void tiffDeform::src_coord_box( )
{
    geoRECT pixel_rect;
    pixel_rect.left = 0 ;
    pixel_rect.top = 0;
    pixel_rect.right = _tiff_file->tif_width;
    pixel_rect.botton = _tiff_file->tif_height ;
    geo_coord( _tiff_file , pixel_rect , _geo_rect_src );
}

double tiffDeform::zone_number( double left_right )
{
    if( _coord_type == 0 )
    {
        return left_right;
    }

    if( left_right/10000000 > 1 )
    {
        return left_right;
    }
    else
    {
        int zone_n = get_zone_num( _proj4_str );
        left_right += ( zone_n * 1000000 );
        return left_right;
    }
}

int tiffDeform::get_zone_num( string proj4_str )
{
    int pos = proj4_str.find("+x_0=");
    string str = proj4_str.substr( pos + 5 );
    return atoi( str.substr( 0 , 2 ).c_str() );
}

double tiffDeform::src_geo_xy( double left_right )
{
    if( _coord_type == 0 )
    {
        return left_right;
    }
    if ( _geo_rect_src.left / 10000000 > 1 )
    {
        return left_right;
    }
    else
    {
        int zone_n = get_zone_num( _proj4_str );
        left_right -= ( zone_n * 1000000 );
        return left_right;
    }
}

void tiffDeform::get_pixel_scale()
{
    if( _coord_type == 0 )//经纬度坐标
    {
        _scaleX_src = _tiff_file->geo_tiff.pixel_scale.scaleX;
        _scaleY_src = _tiff_file->geo_tiff.pixel_scale.scaleY;

        _scaleX_des = ( _geo_rect_des.right - _geo_rect_des.left )/_tiff_file->tif_width;
        _scaleY_des = ( _geo_rect_des.top - _geo_rect_des.botton ) / _tiff_file->tif_height;
    }
    else//平面投影坐标
    {
        _scaleX_src = _tiff_file->geo_tiff.pixel_scale.scaleX;
        _scaleY_src = _tiff_file->geo_tiff.pixel_scale.scaleY;

        _scaleX_des = _scaleX_src;
        _scaleY_des = _scaleY_src;
    }
}

void tiffDeform::to_web_mector( geoRECT&rect , bool b )
{
    /*
      (1)     (2)
        +-----+
        |     |
        |     |
        +-----+
      (3)     (4) 
    */
    _coord_trans.init_proj( _proj4_str.c_str() );

    double x1 ,y1,x2 ,y2,x3 ,y3,x4 ,y4;
    x1 = rect.left;
    y1 = rect.top;
    if ( _coord_type == 0 )
    {
        _coord_trans.longlat_2_xy( x1 , y1 );
    }
    else
    {
        _coord_trans.xy_2_xy( x1 , y1 );
    }

    x2 = rect.right;
    y2 = rect.top;
    if ( _coord_type == 0 )
    {
        _coord_trans.longlat_2_xy( x2 , y2 );
    }
    else
    {
        _coord_trans.xy_2_xy( x2 , y2 );
    }

    x3 = rect.left;
    y3 = rect.botton;
    if ( _coord_type == 0 )
    {
        _coord_trans.longlat_2_xy( x3 , y3 );
    }
    else
    {
        _coord_trans.xy_2_xy( x3 , y3 );
    }

    x4 = rect.right;
    y4 = rect.botton;
    if ( _coord_type == 0 )
    {
        _coord_trans.longlat_2_xy( x4 , y4 );
    }
    else
    {
        _coord_trans.xy_2_xy( x4 , y4 );
    }

    if ( b )
    {
        rect.left = x1 < x3 ? x3 : x1;
        rect.top = y1 > y2 ? y2 : y1;
        rect.right = x2 > x4 ? x4 : x2;
        rect.botton = y3 < y4 ? y4 : y3;
    }
    else
    {
        rect.left = x1 > x3 ? x3 : x1;
        rect.top = y1 < y2 ? y2 : y1;
        rect.right = x2 < x4 ? x4 : x2;
        rect.botton = y3 > y4 ? y4 : y3;
    }
}

void tiffDeform::to_web_mector( geoRECT&rect , bool b , double&x1 , double&y1, double&x2 , double&y2, double&x3 , double&y3, double&x4 , double&y4)
{
    /*
      (1)     (2)
        +-----+
        |     |
        |     |
        +-----+
      (3)     (4) 
    */
    _coord_trans.init_proj( _proj4_str.c_str() );

    //double x1 ,y1,x2 ,y2,x3 ,y3,x4 ,y4;
    x1 = rect.left;
    y1 = rect.top;
    if ( _coord_type == 0 )
    {
        _coord_trans.longlat_2_xy( x1 , y1 );
    }
    else
    {
        _coord_trans.xy_2_xy( x1 , y1 );
    }

    x2 = rect.right;
    y2 = rect.top;
    if ( _coord_type == 0 )
    {
        _coord_trans.longlat_2_xy( x2 , y2 );
    }
    else
    {
        _coord_trans.xy_2_xy( x2 , y2 );
    }

    x3 = rect.left;
    y3 = rect.botton;
    if ( _coord_type == 0 )
    {
        _coord_trans.longlat_2_xy( x3 , y3 );
    }
    else
    {
        _coord_trans.xy_2_xy( x3 , y3 );
    }

    x4 = rect.right;
    y4 = rect.botton;
    if ( _coord_type == 0 )
    {
        _coord_trans.longlat_2_xy( x4 , y4 );
    }
    else
    {
        _coord_trans.xy_2_xy( x4 , y4 );
    }

    if ( b )
    {
        rect.left = x1 < x3 ? x3 : x1;
        rect.top = y1 > y2 ? y2 : y1;
        rect.right = x2 > x4 ? x4 : x2;
        rect.botton = y3 < y4 ? y4 : y3;
    }
    else
    {
        rect.left = x1 > x3 ? x3 : x1;
        rect.top = y1 < y2 ? y2 : y1;
        rect.right = x2 < x4 ? x4 : x2;
        rect.botton = y3 > y4 ? y4 : y3;
    }
}

int tiffDeform::get_src_tag_list()
{
    TIFF_UINT32_T ifd_offset;      //第一个IFD的偏移量
    fseek( _tiff_file->pfile , 0 , SEEK_SET );
    fseek( _tiff_file->pfile ,4 ,SEEK_SET );
    ifd_offset = get4( _tiff_file->pfile , _tiff_file->tiff_byte_order ) ;

    //定位到IFD的位置
    fseek( _tiff_file->pfile , ifd_offset , SEEK_SET );
    //得到IFD的数量
    _de_num = get2( _tiff_file->pfile , _tiff_file->tiff_byte_order );

    de_list = new deInfo[ _de_num ];
    memset( de_list , 0 , _de_num * sizeof( deInfo ) );

    //循环得到DE
    for ( TIFF_UINT16_T i = 0x0000 ; i < _de_num ; i++ )
    {
        fseek( _tiff_file->pfile , ifd_offset + ONE_DE_SIZE * i + 2 , SEEK_SET );//文件指针复原指向
        de_list[i].de.tag = get2( _tiff_file->pfile , _tiff_file->tiff_byte_order );
        de_list[i].de.type = get2( _tiff_file->pfile , _tiff_file->tiff_byte_order );
        de_list[i].de.count = get4( _tiff_file->pfile , _tiff_file->tiff_byte_order );

        //如果是大端字节序并且是short类型，则只会读取四个字节中的前两个字节
        if ( de_list[i].de.type == 3 && _tiff_file->tiff_byte_order == 0x4d4d/*Motor*/ && de_list[i].de.count == 1 )
        {
            de_list[i].de.offset = (TIFF_UINT32_T)get2( _tiff_file->pfile , _tiff_file->tiff_byte_order );
        }
        else
        {
            de_list[i].de.offset = get4( _tiff_file->pfile , _tiff_file->tiff_byte_order );
        }

        //如果是 SHORT 或者 LONG 并且数量为1，则直接存储在Offset中，并不存储地址
        if( ( de_list[i].de.type == 3 || de_list[i].de.type == 4 ) && de_list[i].de.count == 1 )
        {
            de_list[i].data_source = 0 ;
        }
        else
        {
            de_list[i].data_source = 1 ;
        }
    }

    print_tag_info_list();
    return _de_num ;
}

void tiffDeform::print_tag_info_list()
{
    printf( "\n" );
    for ( int i = 0 ; i < _de_num ; i++ )
    {
        char outStr[1024];
        memset( outStr , 0 , 1024 * sizeof( char ) );
        sprintf( outStr , "0x%04x[ %5d %-26s ] , 0x%02x , 0x%04x( %5d ) , 0x%08x , %d \n" 
            , de_list[i].de.tag 
            , de_list[i].de.tag
            , tag_text( de_list[i].de.tag )
            , de_list[i].de.type 
            , de_list[i].de.count
            , de_list[i].de.count
            , de_list[i].de.offset 
            , de_list[i].data_source ) ;
        printf( outStr );
    }
}

const char* tiffDeform::tag_text( int i_tag )
{
    int i = 0 ;
    while ( tag_text_list[i].i_tag != -1 )
    {
        if ( tag_text_list[i].i_tag == i_tag )
        {
            return tag_text_list[i].text;
        }
        i++;
    }

    return "";
}

deInfo* tiffDeform::cts_strip_offsets()
{
    deInfo* temp_de = new deInfo;
    temp_de->de.tag = 273;
    temp_de->de.type = 4;//long
    temp_de->de.count = _new_tiff_height;
    temp_de->de.offset = 0;
    temp_de->data_source = 2;
    TIFF_UINT32_T* mem = new TIFF_UINT32_T[_new_tiff_height];
    memset( mem , 0 , sizeof(TIFF_UINT32_T)*_new_tiff_height );
    temp_de->mem_data = (TIFF_UINT8_T*)mem;
    return temp_de;
}

deInfo* tiffDeform::cts_rows_per_strip()
{
    deInfo* temp_de = new deInfo;
    temp_de->de.tag = 278;
    temp_de->de.type = 3;//short
    temp_de->de.count = 1;
    temp_de->de.offset = 1;
    temp_de->data_source = 0;
    temp_de->mem_data = NULL;
    return temp_de;
}

deInfo* tiffDeform::cts_strip_byte_counts()
{
    deInfo* temp_de = new deInfo;
    temp_de->de.tag = 279;
    temp_de->de.type = 4;//short
    temp_de->de.count = _new_tiff_height;
    temp_de->de.offset = 0;
    temp_de->data_source = 2;
    TIFF_UINT32_T* mem = new TIFF_UINT32_T[_new_tiff_height];
    memset( mem , 0 , sizeof(TIFF_UINT32_T)*_new_tiff_height );
    for ( int i = 0 ; i < _new_tiff_height ; i++ )
    {
        mem[i] = _new_tiff_width * _tiff_file->samples_per_pixel;
    }
    temp_de->mem_data = (TIFF_UINT8_T*)mem;
    return temp_de;
}

deInfo* tiffDeform::cts_line_tag()
{
    deInfo* temp_line_tag_list = new deInfo[3];
    memset( temp_line_tag_list , 0 , sizeof(deInfo) * 3 );
    deInfo* temp = cts_strip_offsets();
    memcpy( temp_line_tag_list , temp , sizeof(deInfo) ) ;
    delete temp;
    temp = cts_rows_per_strip();
    memcpy( temp_line_tag_list + 1 , temp , sizeof(deInfo));
    delete temp;
    temp = cts_strip_byte_counts();
    memcpy( temp_line_tag_list + 2 , temp , sizeof(deInfo));
    delete temp;
    temp = NULL;
    return temp_line_tag_list;
}

int tiffDeform::cts_new_tag_list( )
{
    deInfo* temp_line = cts_line_tag();

    deInfo* temp_de_list = new deInfo[ _de_num ];//tile 4个标签 line 只需要3个标签
    memset( temp_de_list , 0 , sizeof(deInfo)*( _de_num ) );

    int tag_num = 0;
    int j = 0 , k = 0 ;
    for ( int i = 0 ; i < _de_num ; i++ )
    {
        if (   de_list[i].de.tag == 33550 
            || de_list[i].de.tag == 33922
            || de_list[i].de.tag == 34735
            || de_list[i].de.tag == 34736
            || de_list[i].de.tag == 34737)
        {
            continue;
        }

        tag_num ++;
        if ( k < 3 )//line只有三个标签
        {
            if ( de_list[i].de.tag < temp_line[k].de.tag )
            {
                memcpy( temp_de_list + j , de_list + i , sizeof( deInfo ) );
                j++;
            }
            else
            {
                memcpy( temp_de_list + j , temp_line + k , sizeof( deInfo ) );
                j++;
                k++;
                //i--;
            }
        }
        else
        {
            memcpy( temp_de_list + j , de_list + i , sizeof( deInfo ) );
            j++;
        }
        int n = j - 1;
        if ( temp_de_list[n].de.tag == 256 )//ImageWidth
        {
            temp_de_list[n].de.offset = _new_tiff_width;
        }

        if ( temp_de_list[n].de.tag == 257 )//ImageHeight
        {
            temp_de_list[n].de.offset = _new_tiff_height;
        }
    }

    delete[] de_list;
    de_list = NULL;
    de_list = temp_de_list;

    _de_num = tag_num;

    cs_pixel_scale( _scaleX_des , _scaleY_des );
    cs_tie_point( _geo_rect_des.left , _geo_rect_des.top );
    deInfo geo_coord_list[5];
    memset( geo_coord_list , 0 , 5 * sizeof( deInfo ) );
    cs_coord( geo_coord_list );

    mix_proj_de_list( geo_coord_list );

    print_tag_info_list();
    return _de_num;
}

void tiffDeform::mix_proj_de_list( deInfo* coord_list )
{
    deInfo* new_de_list = new deInfo[ _de_num + 5 ];
    memset( new_de_list , 0 , sizeof( deInfo ) * ( _de_num + 5 ) );


    int i_des = 0 ; 
    int i_coord = 0 ;
    if ( _de_num < 5 )
    {
        return ;
    }

    int count = 0;
    for( int i = 0 ; i < _de_num ; i++ )
    {
        if ( de_list[i].de.tag < coord_list[i_coord].de.tag )
        {
            new_de_list[i_des] = de_list[i];
            i_des ++;
            count ++;
        }
        else if ( de_list[i].de.tag >= coord_list[i_coord].de.tag )
        {
            new_de_list[i_des] = coord_list[i_coord];
            i_des++;
            i_coord++;
            count++;
        }
    }

    if ( i_coord < 5 )
    {
        int temp = i_coord;
        for ( int i = temp ; i < 5 ; i++ )
        {
            new_de_list[ i_des ] = coord_list[ i_coord ];
            i_des ++;
            i_coord ++;
            count++;
        }
    }

    delete[] de_list;
    de_list = NULL;

    de_list = new_de_list;
    _de_num = count;
}

TIFF_UINT64_T tiffDeform::double_to_long( double d_ )
{
    union { TIFF_UINT64_T i; double f; } u;
    u.f = d_ ;
    return u.i ;
}

void tiffDeform::cs_pixel_scale( double cx , double cy )
{
    pixel_scale[0] = double_to_long ( cx );
    pixel_scale[1] = double_to_long ( cy );
    pixel_scale[2] = 0 ;
}

void tiffDeform::cs_tie_point( double x , double y )
{
    tie_point[0] = 0;
    tie_point[1] = 0;
    tie_point[2] = 0;

    tie_point[3] = double_to_long( x ) ;
    tie_point[4] = double_to_long( y ) ;
    tie_point[5] = 0;
}

void tiffDeform::cs_coord( deInfo* coord_list )
{
    coord_list[0].de.tag = 33550;//GeoTagPixelScale
    coord_list[0].de.type = 12;//double
    coord_list[0].de.count = 3 ;
    coord_list[0].data_source = 2;//内存数据
    coord_list[0].mem_data = ( TIFF_UINT8_T* )pixel_scale;

    coord_list[1].de.tag = 33922;//GeoTagTiePoint
    coord_list[1].de.type = 12;//double
    coord_list[1].de.count = 6 ;
    coord_list[1].data_source = 2;//内存数据
    coord_list[1].mem_data = ( TIFF_UINT8_T* )tie_point;

    coord_list[2].de.tag = 34735;//GeoTagDirectory
    coord_list[2].de.type = 3;//short
    coord_list[2].de.count = 80;
    coord_list[2].data_source = 2;//内存数据
    coord_list[2].mem_data = ( TIFF_UINT8_T* )geotiff_web_mercator_tag_dir;

    coord_list[3].de.tag = 34736;//GeoDoubleParamsTag
    coord_list[3].de.type = 12;//double
    coord_list[3].de.count = 7;
    coord_list[3].data_source = 2;//内存数据
    coord_list[3].mem_data = ( TIFF_UINT8_T* )geotiff_double_param;

    coord_list[4].de.tag = 34737;//GeoAsciiParamsTag
    coord_list[4].de.type = 2;//ascii
    coord_list[4].de.count = 64;
    coord_list[4].data_source = 2;//内存数据
    coord_list[4].mem_data = ( TIFF_UINT8_T* )geotiff_ascii_param;
}

![复制代码](https://common.cnblogs.com/images/copycode.gif)

main

![复制代码](https://common.cnblogs.com/images/copycode.gif)

#include <stdio.h>
#include <stdlib.h>
#include "MainH.h"
#include "tiffDeform.h"

int main( int argc, char* argv[] )
{
    if ( argc < 4 )
    {
        print_help_info();
        return -1;
    }

    string tiff_path( argv[1] );
    int coord_type = 0;
    string proj4_str = parse_proj4 ( argc , argv , coord_type );

    tiffDeform tiff_deform;
    tiff_deform.open( tiff_path , proj4_str , coord_type );

    string src_tiff = tiff_path;
    string save_name;
    if ( tiff_deform.arrangement() )
    {
        string cmd_sz = get_exe_dir( argv[0] );
        cmd_sz += "Tile2Line.exe \"-t\" ";
        cmd_sz += "\"";
        cmd_sz += tiff_path ;
        cmd_sz += "\"";
        cmd_sz += " ";
        cmd_sz += "\"";
        cmd_sz += argv[2];
        cmd_sz += "\"";
        system( cmd_sz.c_str() );
        src_tiff = deal_tiff_name( argv[2] , tiff_path );
    }

    save_name = new_tiff_name( argv[2] , tiff_path );

    tiff_deform.save( src_tiff  , save_name );

    printf( "转换完成！" );

    return 0;
}

string new_tiff_name( string save_dir ,string src_name )
{
    int pos = src_name.rfind( '\\' );
    string tiff_name = src_name.substr( pos + 1 );
    string temp_name = "\\new_";
    temp_name += tiff_name ;
    return ( save_dir + temp_name ) ;
}

string deal_tiff_name( string save_dir ,string src_name )
{
    int pos = src_name.rfind( '\\' );
    string tiff_name = src_name.substr( pos + 1 );
    string temp_name = "\\line_";
    temp_name += tiff_name ;
    return ( save_dir + temp_name ) ;
}

string get_exe_dir( char* argv )
{
    string str( argv );
    int pos = str.rfind('\\') ;
    return str.substr( 0 , pos + 1 );
}

void print_help_info()
{
    printf("-?/H/h 显示帮助信息 \n");
    printf("参数：[tiff图片完整的路径][保存TIFF的文件夹] [坐标类型 0-经纬度坐标 1-平面投影坐标] [当前tiff图片坐标信息的proj4字符串] \n");
    printf(" 例 ：D:\\beijing.tif D:\\xxx 0 +proj=longlat +ellps=WGS84 +datum=WGS84 +no_defs\n" );
}

string parse_proj4( int argc , char* argv[] ,int&coord_type )
{
    coord_type = atoi( argv[3] );
    string temp_sz;
    for ( int i = 4 ; i < argc ; i++ )
    {
        temp_sz += argv[i];
        temp_sz+=" ";
    }
    return temp_sz.substr( 0 , temp_sz.size() -1 );
}

![复制代码](https://common.cnblogs.com/images/copycode.gif)