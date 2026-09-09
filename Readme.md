1.使用方法:
    创建一个gt20l的对象
    调用里面的方法
2.构造方法:
    class gt20l:
        def __init__(self,spi,cs):
    其中spi是字库的spi对象,cs是字库的cs引脚，类型是Pin
3.gt20l类的方法作用(代码中也有注释,可以看):

get_font(self,addr,byte): addr是字的地址，byte是读取的字节数,返回该地址的字模数据，内部调用，对于用户用处不大

get_gb2312_font(self,gb):gb是一个列表,一共有两个元素,gb[0]是要获取的gb2312字模的高八位(int),gb[1]是要获取的gb2312字模的低八位(int)，该方法用于获取一个gb2312编码字符的字模,返回值是点阵数据

get_gb2312_efont(self,gb):与上相同,用于获取国标扩展字符的字模，但是其中的gb为该字符的内码(详见用户手册第21页)，返回值是点阵数据

get_57ascll(self,let):获取5x7的ascll字符的字模,其中let为要获取字模的ascll字符(str),返回值是点阵数据

下面的方法和get_57ascll方法使用基本相同，只是返回值不一样，在这里就不赘述了

get_816ascll(self,let):获取8x16ascll字模

get_16Arial_ascll(self,let):获取16点阵不等宽ascll字模

get_816bold_ascll(self,let):获取8x16粗体ascll字模

get_TimesNewRoman_ascll(self,let):获取16点阵白头不等宽ascll字模

比方说，我要获取"a"的8x16字模，那么我可以:
gt20l=gt20l.gt20l(spi,cs)
data=gt20l.get_816ascll('a')
此时的data就是a的点阵数据

4.常见问题
    字模全部是0x00或者0xff：
	字库没有连接好或者是spi引脚定义错误

    NameError: local variable referenced before assignment：
	输入的内容不合法,比如说函数要求输入的是gb2312编码的，你输入的不是，就可能会有这种问题
