# 游戏大厅五子棋

## 客户端部分代码

packdef.h

```cpp
#pragma once

#include<memory.h>

#define _DEF_BUFFER         (4096)
#define _DEF_CONTENT_SIZE	(1024)
#define _MAX_SIZE           (40)
#define _MAX_PATH           (260)

//自定义协议   先写协议头 再写协议结构
//登录 注册 获取好友信息 添加好友 聊天 发文件 下线请求
#define _DEF_PACK_BASE	(10000)
#define _DEF_PACK_COUNT (100)

//注册
#define _DEF_PACK_REGISTER_RQ	(_DEF_PACK_BASE + 0 )
#define _DEF_PACK_REGISTER_RS	(_DEF_PACK_BASE + 1 )
//登录
#define _DEF_PACK_LOGIN_RQ	(_DEF_PACK_BASE + 2 )
#define _DEF_PACK_LOGIN_RS	(_DEF_PACK_BASE + 3 )

//p.s. 客户端是聊天的协议 服务端是音视频的协议 2025.12.26
//游戏头文件在下面

//返回的结果
//注册请求的结果 注册带则会昵称
#define tel_is_exist		(0)
#define register_success	(1)
#define name_is_exist        (2)
//登录请求的结果
#define user_not_exist		(0)
#define password_error		(1)
#define login_success		(2)
//添加好友
#define no_this_user        (0)
#define user_refuse         (1)
#define user_offline        (2)
#define add_success         (3)

typedef int PackType;

//协议结构
//注册
typedef struct STRU_REGISTER_RQ
{
	STRU_REGISTER_RQ():type(_DEF_PACK_REGISTER_RQ)
	{
		memset( tel  , 0, sizeof(tel));
		memset( name  , 0, sizeof(name));
		memset( password , 0, sizeof(password) );
	}
	//需要手机号码 , 密码, 昵称
	PackType type;
	char tel[_MAX_SIZE];
	char name[_MAX_SIZE];
	char password[_MAX_SIZE];

}STRU_REGISTER_RQ;

typedef struct STRU_REGISTER_RS
{
	//回复结果
	STRU_REGISTER_RS(): type(_DEF_PACK_REGISTER_RS) , result(register_success)
	{
	}
	PackType type;
	int result;

}STRU_REGISTER_RS;

//登录
typedef struct STRU_LOGIN_RQ
{
	//登录需要: 手机号 密码 
	STRU_LOGIN_RQ():type(_DEF_PACK_LOGIN_RQ)
	{
		memset( tel , 0, sizeof(tel) );
		memset( password , 0, sizeof(password) );
	}
	PackType type;
	char tel[_MAX_SIZE];
	char password[_MAX_SIZE];

}STRU_LOGIN_RQ;

typedef struct STRU_LOGIN_RS
{
	// 需要 结果 , 用户的id
	STRU_LOGIN_RS(): type(_DEF_PACK_LOGIN_RS) , result(login_success),userid(0)
	{
        memset( name , 0, sizeof(name) );
	}
	PackType type;
	int result;
	int userid;
    char name[_MAX_SIZE];

}STRU_LOGIN_RS;

/*------------------------------------游戏相关------------------------------------*/

#define DEF_PACK_JOIN_ZONE      (_DEF_PACK_BASE + 4 )
#define DEF_PACK_LEAVE_ZONE     (_DEF_PACK_BASE + 5 )

enum ENUM_PLAY_ZONE{Five_in_Line = 0x10, E_L_S, D_D_Z};
//加入专区
struct STRU_JOIN_ZONE{
    //解决这是什么包 谁加入哪个专区
    STRU_JOIN_ZONE():type(DEF_PACK_JOIN_ZONE),userid(0),zoneid(0){

    }
    PackType type;
    int userid;
    int zoneid;
};
//退出专区
struct STRU_LEAVE_ZONE{
    //解决这是什么包 谁退出哪个区
    STRU_LEAVE_ZONE():type(DEF_PACK_LEAVE_ZONE),userid(0){

    }
    PackType type;
    int userid;
};

//专区内每个房间人数————和视频里的不一样 注意
#define DEF_ZONE_ROOM_INFO ( _DEF_PACK_BASE + 10 )
#define DEF_ZONE_INFO_RQ   ( _DEF_PACK_BASE + 11 )

#define DEF_ZONE_ROOM_COUNT 121
//请求 专区每个房间人数
struct STRU_ZONE_INFO_RQ{

    STRU_ZONE_INFO_RQ():type( DEF_ZONE_INFO_RQ )
        ,zoneid(0)
    {

    }
    PackType type;
    int zoneid;
};

struct STRU_ZONE_ROOM_INFO{
    //解决这是什么包 谁退出哪个区
    STRU_ZONE_ROOM_INFO():type( DEF_ZONE_ROOM_INFO )
        ,zoneid(0)
    {
        memset( roomInfo, 0, sizeof( roomInfo ));
    }
    PackType type;
    int zoneid;
    int roomInfo[ DEF_ZONE_ROOM_COUNT ];
};

#define DEF_JOIN_ROOM_RQ    (_DEF_PACK_BASE + 6 )

//加入房间
struct STRU_JOIN_ROOM_RQ{
    //解决这是什么包 谁加入哪个房间
    STRU_JOIN_ROOM_RQ():type(DEF_JOIN_ROOM_RQ),userid(0),roomid(0){

    }
    PackType type;
    int userid;
    int roomid;
};//发给服务器 服务器会同步房间成员信息

//房间 为了避免0 出现歧义 房间号是0 还是没有初始化 把0让出来 1-120

#define DEF_JOIN_ROOM_RS    (_DEF_PACK_BASE + 7 )
enum ENUM_ROOM_STATUS{ _host, _player, _watcher };//房主 玩家 观战者
//加入房间回复
struct STRU_JOIN_ROOM_RS{
    //解决这是什么包 谁 哪个房间 叫什么名字
    STRU_JOIN_ROOM_RS():type( DEF_JOIN_ROOM_RS ),
        userid(0),
        roomid(0),
        status(_host),
        result(1){

    }
    PackType type;
    int userid;
    int roomid;
    int status;
    int result;// 0 fail 1 success
};

#define DEF_ROOM_MEMBER     ( _DEF_PACK_BASE + 8 )
//房间成员
struct STRU_ROOM_MEMBER{
    //解决这是什么包 谁 哪个房间 叫什么名字
    STRU_ROOM_MEMBER():type( DEF_ROOM_MEMBER ), userid(0), status(_player){
        memset( name, 0, sizeof(name) );
    }
    PackType type;
    int userid;
    //加上身份
    int status;
    char name[_MAX_SIZE];
};


#define DEF_LEAVE_ROOM_RQ   ( _DEF_PACK_BASE + 9 )
//退出房间
struct STRU_LEAVE_ROOM_RQ{
    //解决这是什么包 谁 退出了房间
    STRU_LEAVE_ROOM_RQ():type( DEF_LEAVE_ROOM_RQ ),
        userid(0),status(_player),roomid(0){

    }
    PackType type;
    int userid;
    int status;
    int roomid;
};//会被转发 如果自己不是房主 房主退出 自己也跟着退出

```

ckernel.h

```cpp
#ifndef CKERNEL_H
#define CKERNEL_H

#include <QObject>
#include "INetMediator.h"
#include "packdef.h"
#include <vector>

#include "maindialog.h"
#include "logindialog.h"
#include "fiveinlinezone.h"
#include "roomdialog.h"

//成员函数指针类型
class CKernel;
typedef void (CKernel::*PFUN)( unsigned int lSendIP , char* buf , int nlen );

//单例 最简单 静态的
class CKernel : public QObject
{
    Q_OBJECT
public:
    static CKernel * GetInstance(){
        static CKernel kernel;
        return &kernel;
    }

public slots:
    void DestroyInstance();
    void SendData( char * buf, int nlen );
    /*
     * 窗口处理
     * 2025.12.24: add slot_logincommit and slotregistercommit
     * 2025.12.26: add slot_joinZone
     */
    void slot_loginCommit(QString tel, QString password);
    void slot_registerCommit(QString tel, QString password, QString name);
    void slot_leaveZone();
    void slot_joinZone(int zoneid);
    /*
     * time 2025.12.27
     * slot leave room
     */
    void slot_leaveRoom();
    /*
     * time 2025.12.27
     * definite slot_joinRoom(int)
     */
    void slot_joinRoom(int roomid);


    void slot_ReadyData( unsigned int lSendIP , char* buf , int nlen );
    //网络处理
    void slot_dealloginRs( unsigned int lSendIP , char* buf , int nlen );
    void slot_dealregisterRs( unsigned int lSendIP , char* buf , int nlen );
    void slot_dealJoinRoomRs( unsigned int lSendIP , char* buf , int nlen );
    void slot_dealRoomMemberRq( unsigned int lSendIP , char* buf , int nlen );
    void slot_dealLeaveRoomRq(unsigned int lSendIP, char *buf, int nlen);

private:
    void setNetPackFunMap();
    void ConfigSet();
    explicit CKernel(QObject *parent = nullptr);
    ~CKernel(){  }
    CKernel(const CKernel& kernel){};
    CKernel& operator = (const CKernel& kernel){
        return *this;
    }
    //成员属性 网络 ui类对象
    MainDialog * m_mainDialog;
    LoginDialog * m_loginDialog;
    FiveInLineZone * m_fiveInLineZone;
    RoomDialog * m_roomDialog;

    INetMediator * m_client;

    //协议映射表 协议头与处理函数的处理关系
    std::vector<PFUN> m_netPackFunMap;

    //个人信息
    int m_id;
    int m_roomid;
    int m_zoneid;
    bool m_isHost;
    QString m_userName;
    char m_serverIP[20];
};

#endif // CKERNEL_H

```

roomdialog.h

```cpp
#ifndef ROOMDIALOG_H
#define ROOMDIALOG_H

#include <QDialog>
#include <QCloseEvent>

namespace Ui {
class RoomDialog;
}

class RoomDialog : public QDialog
{
    Q_OBJECT
signals:
    void SIG_close();
public:
    explicit RoomDialog(QWidget *parent = nullptr);
    ~RoomDialog();

    void setInfo( int roomid );
    /*
     * time 2025.12.28
     * sethostinfo setplayerinfo
     */
    void setUserStatus( int status );
    void setHostInfo( int id, QString name );
    void setPlayerInfo( int id, QString name);
    void clearRoom();
    void playerLeave(int id);

    void closeEvent(QCloseEvent * event);

private:
    Ui::RoomDialog *ui;

    int m_roomid;
    /*
     * time 2025.12.28
     * list<int> m_roomUserList
     */
    std::list<int> m_roomUserList;
    //状态
    int m_status;
};

#endif // ROOMDIALOG_H

```

ckernel.cpp

```cpp
#include "ckernel.h"
#include "QDebug"
#include "TcpClientMediator.h"
#include <QMessageBox>
#include "md5.h"

#include <QSettings>
#include <QCoreApplication>
#include <QFileInfo>

//获得md5函数
//1_1234
//EA135E06CD37AB7E304E1DC440C93EA2
//结果：ea135e06cd37ab7e304e1dc440c93ea2
//验证
#define MD5_KEY 1234
static std::string getMD5(QString val){
    QString tmp = QString("%1_%2").arg(val).arg(MD5_KEY);
    MD5 md( tmp.toStdString() );
    return md.toString();
}



//宏定义封装 映射偏移
#define NetPackMap( a ) m_netPackFunMap[ (a) - _DEF_PACK_BASE ]

//协议对应关系
void CKernel::setNetPackFunMap()
{
    //太长了 使用宏定义进行封装
    //m_netPackFunMap[_DEF_PACK_LOGIN_RS - _DEF_PACK_BASE ] = &CKernel::slot_dealloginRs;

    NetPackMap(_DEF_PACK_LOGIN_RS) = &CKernel::slot_dealloginRs;
    NetPackMap(_DEF_PACK_REGISTER_RS) = &CKernel::slot_dealregisterRs;
    /*
     * time 2025.12.27
     *
     */
    NetPackMap(DEF_JOIN_ROOM_RS) = &CKernel::slot_dealJoinRoomRs;
    /*
     * time 2025.12.28
     * room member
     */
    NetPackMap(DEF_ROOM_MEMBER) = &CKernel::slot_dealRoomMemberRq;
    /*
     * time 2025.12.29
     *
     */
    NetPackMap(DEF_LEAVE_ROOM_RQ) = &CKernel::slot_dealLeaveRoomRq;
}



void CKernel::ConfigSet()
{
    //获取配置文件的信息以及设置
    //.ini配置文件
    //[net]组名 groupname
    //key = value

    //ip默认
    strcpy( m_serverIP, _DEF_SERVER_IP );
    //设置和获取配置文件 有还是没有 配置文件在哪里？设置和exe同一级的目录
    QString path = QCoreApplication::applicationDirPath() + "/config.ini";
    QFileInfo info(path);
    if( info.exists() ){
        //存在
        QSettings setting( path, QSettings::IniFormat, nullptr );
        //[net]写入组
        setting.beginGroup("net");
        QVariant var = setting.value( "ip" );
        QString strip = var.toString();
        if( !strip.isEmpty() ){
            strcpy( m_serverIP, strip.toStdString().c_str() );
        }
        setting.endGroup();
    }
    else{
        //不存在
        QSettings setting( path, QSettings::IniFormat, nullptr );
        //[net]写入组
        setting.beginGroup("net");
        setting.setValue( "ip", QString::fromStdString(m_serverIP) );
        setting.endGroup();
    }
    qDebug()<< "ip: " << m_serverIP;
}

void CKernel::DestroyInstance()
{
    qDebug()<<__func__;
    if(m_client){
        m_client->CloseNet();
        delete m_client;
        m_client = nullptr;
    }
    delete m_mainDialog;
    delete m_loginDialog;
}

void CKernel::slot_loginCommit(QString tel, QString password)
{
    //加密——挖坑——完成
    //封包
    STRU_LOGIN_RQ rq;
    strcpy( rq.tel, tel.toStdString().c_str() );

    //qDebug()<<password<<"MD5"<<getMD5( password ).c_str();

    strcpy( rq.password, getMD5( password ).c_str() );
    //发送
    SendData( (char * )&rq, sizeof(rq) );
}

void CKernel::slot_registerCommit(QString tel, QString password, QString name)
{
    //加密——挖坑——完成
    //封包
    STRU_REGISTER_RQ rq;
    strcpy( rq.tel, tel.toStdString().c_str() );
    strcpy( rq.password, getMD5( password ).c_str() );
    //兼容中文
    std::string strName = name.toStdString();
    strcpy( rq.name, strName.c_str() );
    //发送
    SendData( (char *)&rq, sizeof(rq) );
}
/*
 * time 2025.12.27
 * refactor slot leave zone
 */
void CKernel::slot_leaveZone()
{
    //成员属性修改
    m_zoneid = 0;
    //请求
    STRU_LEAVE_ZONE rq;
    rq.userid = m_id;
    SendData( (char *)&rq, sizeof(rq) );
    //ui跳转
    m_mainDialog->show();
}
/*
 * author jssun
 * time 2025.12.26: add a definition of slot_joinZone
 */
 //提交加入分区
void CKernel::slot_joinZone(int zoneid)
{
    /*
     * time 2025.12.27
     * member changes the property
     */
    m_zoneid = zoneid;

    //请求包
    STRU_JOIN_ZONE rq;
    rq.userid = m_id;
    rq.zoneid = zoneid;

    SendData( (char *)&rq, sizeof(rq) );
    /*
     * time 2025.12.27
     * show the five in line zone
     */
    switch( zoneid ){
    case Five_in_Line:
        m_fiveInLineZone->show();
        break;
    }
}
/*
 * time 2025.12.29
 * leave room
 */
void CKernel::slot_leaveRoom()
{
    //这个人主动离开
    STRU_LEAVE_ROOM_RQ rq;
    rq.status = m_isHost?_host:_player;
    rq.userid = m_id;
    rq.roomid = m_roomid;
    SendData((char *)&rq, sizeof(rq));
    //界面
    m_roomDialog->clearRoom();
    m_roomDialog->hide();
    m_fiveInLineZone->show();
    //后台数据
    m_roomid = 0;
    m_isHost = false;
}
/*
 * time 2025.12.27
 * refactor slot_joinRoom
 */
//提交加入房间
void CKernel::slot_joinRoom(int roomid)
{
    /*
     * time 2025.12.27
     * judge if the roomid != 0
     */
    if( m_roomid != 0 ){
        QMessageBox::about( nullptr, "Oops", "已经在房间，无法加入");
        return;
    }
    //加入成功后隐藏
    //m_fiveInLineZone->hide();
    //提交请求
    STRU_JOIN_ROOM_RQ rq;
    rq.userid = m_id;
    rq.roomid = roomid;

    SendData( (char *)&rq, sizeof(rq) );
}

//接收处理
void CKernel::slot_ReadyData(unsigned int lSendIP, char *buf, int nlen)
{
    //协议映射表
    PackType type = *(int *)buf;    //*(int *) 按照整型取数据
    if(type >= _DEF_PACK_BASE
        && type < _DEF_PACK_BASE + _DEF_PACK_COUNT )
    {
        //根据协议头跳转到对应函数
        //不推荐switch 每此都要增加，修改带着swich
        //协议映射处理
        PFUN pf = NetPackMap( type );
        //执行函数
        (this->*pf)( lSendIP,buf,nlen );

    }
    //要记得回收buf
    delete[] buf;
}

void CKernel::slot_dealloginRs(unsigned int lSendIP, char *buf, int nlen)
{
    qDebug()<<__func__;
    //拆包
    STRU_LOGIN_RS * rs = (STRU_LOGIN_RS *)buf;
    //根据不同结果显示
    switch( rs->result ){
    case user_not_exist:
        QMessageBox::about( m_loginDialog, "Oops", "user not exist login failed");
        break;
    case password_error:
        QMessageBox::about( m_loginDialog, "Oops", "password error login failed");
        break;
    case login_success:
        //ui切换
        m_loginDialog->hide();
        m_mainDialog->show();
        //存储
        m_id = rs->userid;
        m_userName = QString::fromStdString( rs->name );
        break;
    default:
        QMessageBox::about( m_loginDialog, "Oops", "what the fuck did you fucking did?");
        break;
    }
}

void CKernel::slot_dealregisterRs(unsigned int lSendIP, char *buf, int nlen)
{
    qDebug()<<__func__;
    //解析数据包
    STRU_REGISTER_RS * rs = (STRU_REGISTER_RS * )buf;
    //根据结果弹窗
    switch(rs->result){
    case tel_is_exist:
        QMessageBox::about(this->m_loginDialog, "Register Oops", "tel is exist");
        break;
    case name_is_exist:
        QMessageBox::about(this->m_loginDialog, "Register Oops", "name is exist");
        break;
    case register_success:
        QMessageBox::about(this->m_loginDialog, "Register Yeah", "Register SUCCESSFULLY~~");
        break;
    default:
        QMessageBox::about(this->m_loginDialog, "Register Oops", "What the fuck did you fucking did?");
        break;
    }
}
//加入房间回复处理
void CKernel::slot_dealJoinRoomRs(unsigned int lSendIP, char *buf, int nlen)
{
    //拆包
    STRU_JOIN_ROOM_RS * rs = (STRU_JOIN_ROOM_RS *)buf;
    if( rs->result == 0){
        QMessageBox::about( nullptr, "Oops", "加入房间失败" );//专区要具体
        return;
    }
    if( rs->status == _host ){
        m_isHost = true;
    }
    m_roomid = rs->roomid;
    //成功跳转 成员赋值
    m_fiveInLineZone->hide();
    m_roomDialog->setInfo( m_roomid );
    m_roomDialog->show();
    //问题、未来扩展游戏区域不同怎么显示和隐藏
}
/*
 * time 2025.12.28
 * dealRoomMemberRq
 */
void CKernel::slot_dealRoomMemberRq(unsigned int lSendIP, char *buf, int nlen)
{
    //拆包 别人给你发 自己给自己发
    STRU_ROOM_MEMBER *rq = (STRU_ROOM_MEMBER *)buf;
    //设计的时候要加个身份——加身份了
    if( rq->status == _host ){
        m_roomDialog->setHostInfo( rq->userid,
                                  QString::fromStdString(rq->name));
    }
    if( rq->status == _player ){
        m_roomDialog->setPlayerInfo( rq->status,
                                  QString::fromStdString(rq->name));
    }

    m_roomDialog->setUserStatus( m_isHost?_host:_player);
}
/*
 * time 2025.12.29
 */
void CKernel::slot_dealLeaveRoomRq(unsigned int lSendIP, char *buf, int nlen){
    //拆包
    STRU_LEAVE_ROOM_RQ * rq = (STRU_LEAVE_ROOM_RQ *)buf;
    if( rq->status == _host ){
        //界面
        m_roomDialog->clearRoom();
        m_roomDialog->hide();
        m_fiveInLineZone->show();
        //后台数据
        m_roomid = 0;
        m_isHost = false;
    }
    else{
        m_roomDialog->playerLeave(rq->userid);
    }
    //
}


void CKernel::SendData(char *buf, int nlen)
{
    m_client->SendData( 0, buf, nlen );
}

CKernel::CKernel(QObject *parent)
    : QObject{parent}, m_netPackFunMap(_DEF_PACK_COUNT, 0)
    ,m_id(0), m_roomid(0), m_zoneid(0),m_isHost(false)
{
    ConfigSet();
    setNetPackFunMap();
/*-----------------------------------------------------------------------------*/
    m_mainDialog = new MainDialog;

    connect( m_mainDialog, SIGNAL( SIG_close() ),
            this, SLOT(DestroyInstance()) );
    //p.s.如果析构函数里也写了destroyinstance函数 先调用析构的然后再走到connect这里

    /*
     * time: 2025.12.26
     * connect: maindialog five in row push button
     */
    connect( m_mainDialog, SIGNAL( SIG_joinZone(int) ),
            this, SLOT(slot_joinZone(int)) );

    //m_mainDialog->show();
/*-----------------------------------------------------------------------------*/
    //show register & login window
    m_loginDialog = new LoginDialog;

    /*
     * time:2025.12.24
     * connect login, register, close
     */
    connect( m_loginDialog, SIGNAL(SIG_close()),
            this, SLOT(DestroyInstance()) );
    connect( m_loginDialog, SIGNAL(SIG_loginCommit( QString, QString )),
            this, SLOT(slot_loginCommit( QString, QString )) );
    connect( m_loginDialog, SIGNAL(SIG_registerCommit( QString, QString, QString )),
            this, SLOT(slot_registerCommit( QString, QString, QString )) );

    m_loginDialog->show();
/*-----------------------------------------------------------------------------*/

    m_fiveInLineZone = new FiveInLineZone;
    //m_fiveInLineZone->show();
    /*
     * time 2025.12.27
     * connect five in line zone when you click the button
     */
    connect( m_fiveInLineZone, SIGNAL(SIG_joinRoom(int))
            , this, SLOT(slot_joinRoom(int)) );
/*----------------leave zone connection----------------------------------------*/
    connect( m_fiveInLineZone, SIGNAL(SIG_close())
            , this, SLOT(slot_leaveZone()) );


    m_roomDialog = new RoomDialog;
    /*
     * time 2025.12.29
     */
    connect(m_roomDialog, SIGNAL(SIG_close()),
            this, SLOT(slot_leaveRoom()));
    //m_roomDialog->show();

/*-----------------------------------------------------------------------------*/
    m_client = new TcpClientMediator;
    m_client->OpenNet( m_serverIP, _DEF_TCP_PORT );

    connect(m_client, SIGNAL(SIG_ReadyData(uint,char*,int))
            , this, SLOT(slot_ReadyData(uint,char*,int)));

    //模拟连接服务器 发送数据包
    //STRU_LOGIN_RQ rq;//这个位置用的请求不能定义字符串，不可以，因为string用的堆区空间
    //这个senddata相当于copy，拷贝连续空间，给的首地址，拷贝sizeof这么多，
    //故，这个区域一定是连续的，不能定义string，qstring这种
    //m_client->SendData(0,(char*)&rq,sizeof(rq));



}

```

roomdialog.cpp

```cpp
#include "roomdialog.h"
#include "ui_roomdialog.h"
#include <QMessageBox>
#include "packdef.h"

RoomDialog::RoomDialog(QWidget *parent)
    : QDialog(parent)
    , ui(new Ui::RoomDialog), m_roomid(0),m_status(_player)
{
    ui->setupUi(this);

    /*
     * time 2025.12.28
     * make push button uncheckable
     */
    ui->pb_start->setEnabled( false );
    //关于开局的按键 可以是kernel判断 然后给room发送 可以点击开局

    //点击开局 返回开局信号



}

RoomDialog::~RoomDialog()
{
    delete ui;
}

void RoomDialog::setInfo(int roomid)
{
    m_roomid = roomid;

    QString txt = QString("五子棋-%1").arg(roomid, 2, 10, QChar('0'));
    this->setWindowTitle( txt );
}
/*
 * time 2025.12.28
 * set user status
 */
//#include "packdef.h"
void RoomDialog::setUserStatus(int status)
{
    m_status = status;
    //只有房主可以点开局
    if( m_status == _host ){

    }
}
/*
 * time 2025.12.28
 * the definition of setHostInfo
 * 设置房主信息
 */
void RoomDialog::setHostInfo(int id, QString name)
{
    ui->lb_player1_name->setText( name );
    ui->lb_icon_player1->setPixmap(QPixmap(":/images/icon_rui"));
    m_roomUserList.push_back( id );
}
/*
 *time 2025.12.28
 *the definition of setplayerinfo
 *设置玩家信息
 */
void RoomDialog::setPlayerInfo(int id, QString name)
{
    ui->lb_player2_name->setText( name );
    ui->lb_icon_player2->setPixmap(QPixmap(":/images/icon_rui"));
    m_roomUserList.push_back( id );
}

void RoomDialog::clearRoom()
{
    //ui
    ui->lb_player1_name->setText( "等待加入" );
    ui->lb_player2_name->setText( "等待加入" );
    ui->lb_icon_player1->setPixmap(QPixmap(":/Resource2/icon/slotwait.jpg"));
    ui->lb_icon_player2->setPixmap(QPixmap(":/Resource2/icon/slotwait.jpg"));

    //游戏界面清空

    //聊天窗口清空

    //后台数据
    m_roomid = 0;
    m_roomUserList.clear();
    m_status = _player;
}

void RoomDialog::playerLeave(int id)
{
    //ui
    ui->lb_player2_name->setText("等待加入");
    ui->lb_icon_player2->setPixmap(QPixmap(":/Resource2/icon/slotwait.jpg"));
    //data
    for(auto ite = m_roomUserList.begin();ite!=m_roomUserList.end();++ite){
        if(id == *ite){
            ite = m_roomUserList.erase(ite);
            break;
        }
    }
}
//离开房间
void RoomDialog::closeEvent(QCloseEvent *event)
{
    if(QMessageBox::question( this, "exit", "are you sure?")
        == QMessageBox::Yes){
        Q_EMIT SIG_close();
        event->accept();
    }
    else{
        event->ignore();
    }
}

```

## 服务端部分代码

clogic.h

```cpp
#ifndef CLOGIC_H
#define CLOGIC_H

#include"TCPKernel.h"

class CLogic
{
public:
    CLogic( TcpKernel* pkernel ):m_roomUserList( 121 )
    {
        m_pKernel = pkernel;
        m_sql = pkernel->m_sql;
        m_tcp = pkernel->m_tcp;

        //initial mutex
        pthread_mutex_init( &m_roomListMutex, NULL );
    }
public:
    //设置协议映射
    void setNetPackMap();
    /************** 发送数据*********************/
    void SendData( sock_fd clientfd, char*szbuf, int nlen )
    {
        m_pKernel->SendData( clientfd ,szbuf , nlen );
    }
    /************** 网络处理 *********************/
    //注册
    void RegisterRq(sock_fd clientfd, char*szbuf, int nlen);
    //登录
    void LoginRq(sock_fd clientfd, char*szbuf, int nlen);

    /*
     * time 2025.12.26
     * add JoinZoneRq
     */
    //加入分区
    void JoinZoneRq( sock_fd clientfd, char*szbuf, int nlen );
    /*
     * time 2025.12.27
     * LeaveZoneRq joinroomrq
    */
    void LeaveZoneRq( sock_fd clientfd, char*szbuf, int nlen );
    void JoinRoomRq( sock_fd clientfd, char*szbuf, int nlen );
    /*
     * time 2025.12.29
     * 离开房间
     */
    void LeaveRoomRq( sock_fd clientfd, char*szbuf, int nlen );

    /*******************************************/

private:
    TcpKernel* m_pKernel;
    CMysql * m_sql;
    Block_Epoll_Net * m_tcp;

    MyMap<int, UserInfo *> m_mapFdToUserInfo;
    vector<list<int> >m_roomUserList;
    pthread_mutex_t m_roomListMutex;
};

#endif // CLOGIC_H

```

packdef.h

```cpp
#pragma once

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <ctype.h>
#include <sys/epoll.h>
#include <pthread.h>
#include <signal.h>
#include <errno.h>
#include "err_str.h"
#include <malloc.h>

#include<iostream>
#include<map>
#include<list>


//边界值
#define _DEF_SIZE           45
#define _DEF_BUFFERSIZE     1000
#define _DEF_PORT           8000
#define _DEF_SERVERIP       "0.0.0.0"
#define _DEF_LISTEN         128
#define _DEF_EPOLLSIZE      4096
#define _DEF_IPSIZE         16
#define _DEF_COUNT          10
#define _DEF_TIMEOUT        10
#define _DEF_SQLIEN         400
#define TRUE                true
#define FALSE               false



/*-------------数据库信息-----------------*/
#define _DEF_DB_NAME    "PlayHall"
#define _DEF_DB_IP      "localhost"
#define _DEF_DB_USER    "root"
#define _DEF_DB_PWD     "12345"
/*--------------------------------------*/
#define _MAX_PATH           (260)
#define _DEF_BUFFER         (4096)
#define _DEF_CONTENT_SIZE	(1024)
#define _MAX_SIZE           (40)

//自定义协议   先写协议头 再写协议结构
//登录 注册 获取好友信息 添加好友 聊天 发文件 下线请求
#define _DEF_PACK_BASE	(10000)
#define _DEF_PACK_COUNT (100)

//注册
#define _DEF_PACK_REGISTER_RQ	(_DEF_PACK_BASE + 0 )
#define _DEF_PACK_REGISTER_RS	(_DEF_PACK_BASE + 1 )
//登录
#define _DEF_PACK_LOGIN_RQ	(_DEF_PACK_BASE + 2 )
#define _DEF_PACK_LOGIN_RS	(_DEF_PACK_BASE + 3 )


//返回的结果
//注册请求的结果
#define tel_is_exist		(0)
#define register_success	(1)
#define name_is_exist       (2)
//登录请求的结果
#define user_not_exist		(0)
#define password_error		(1)
#define login_success		(2)


typedef int PackType;

//协议结构
//注册
typedef struct STRU_REGISTER_RQ
{
    STRU_REGISTER_RQ():type(_DEF_PACK_REGISTER_RQ)
    {
        memset( tel  , 0, sizeof(tel));
        memset( name  , 0, sizeof(name));
        memset( password , 0, sizeof(password) );
    }
    //需要手机号码 , 密码, 昵称
    PackType type;
    char tel[_MAX_SIZE];
    char name[_MAX_SIZE];
    char password[_MAX_SIZE];

}STRU_REGISTER_RQ;

typedef struct STRU_REGISTER_RS
{
    //回复结果
    STRU_REGISTER_RS(): type(_DEF_PACK_REGISTER_RS) , result(register_success)
    {
    }
    PackType type;
    int result;

}STRU_REGISTER_RS;

//登录
typedef struct STRU_LOGIN_RQ
{
    //登录需要: 手机号 密码
    STRU_LOGIN_RQ():type(_DEF_PACK_LOGIN_RQ)
    {
        memset( tel , 0, sizeof(tel) );
        memset( password , 0, sizeof(password) );
    }
    PackType type;
    char tel[_MAX_SIZE];
    char password[_MAX_SIZE];

}STRU_LOGIN_RQ;

typedef struct STRU_LOGIN_RS
{
    // 需要 结果 , 用户的id
    STRU_LOGIN_RS(): type(_DEF_PACK_LOGIN_RS) , result(login_success),userid(0)
    {
        memset( name , 0, sizeof(name) );
    }
    PackType type;
    int result;
    int userid;
    char name[_MAX_SIZE];

}STRU_LOGIN_RS;

typedef struct UserInfo
{
    UserInfo()
    {
        m_sockfd = 0;
        m_id = 0;
        m_roomid = 0;
        m_zoneid = 0;
        memset(m_userName, 0, _MAX_SIZE);
        //m_videofd = 0;
        //m_audiofd = 0;
    }
    int m_sockfd;
    int m_id;
    int m_roomid;
    int m_zoneid;//area id
    char m_userName[_MAX_SIZE];

    //int m_videofd;
    //int m_audiofd;
}UserInfo;

/*------------------------------------游戏相关------------------------------------*/

#define DEF_PACK_JOIN_ZONE      (_DEF_PACK_BASE + 4 )
#define DEF_PACK_LEAVE_ZONE     (_DEF_PACK_BASE + 5 )

enum ENUM_PLAY_ZONE{Five_in_Line = 0x10, E_L_S, D_D_Z};
//加入专区
struct STRU_JOIN_ZONE{
    //解决这是什么包 谁加入哪个专区
    STRU_JOIN_ZONE():type(DEF_PACK_JOIN_ZONE),userid(0),zoneid(0){

    }
    PackType type;
    int userid;
    int zoneid;
};
//退出专区
struct STRU_LEAVE_ZONE{
    //解决这是什么包 谁退出哪个区
    STRU_LEAVE_ZONE():type(DEF_PACK_LEAVE_ZONE),userid(0){

    }
    PackType type;
    int userid;
};

//专区内每个房间人数————和视频里的不一样 注意
#define DEF_ZONE_ROOM_INFO ( _DEF_PACK_BASE + 10 )
#define DEF_ZONE_ROOM_COUNT 121
struct STRU_ZONE_ROOM_INFO{
    //解决这是什么包 谁退出哪个区
    STRU_ZONE_ROOM_INFO():type( DEF_ZONE_ROOM_INFO ){
        memset( roomInfo, 0, sizeof( roomInfo ));
    }
    PackType type;
    int roomInfo[ DEF_ZONE_ROOM_COUNT ];
};
/*p.s.上面这个只有这些还不够，还要有询问专区的包...*/

#define DEF_JOIN_ROOM_RQ    (_DEF_PACK_BASE + 6 )

//加入房间
struct STRU_JOIN_ROOM_RQ{
    //解决这是什么包 谁加入哪个房间
    STRU_JOIN_ROOM_RQ():type(DEF_JOIN_ROOM_RQ),userid(0),roomid(0){

    }
    PackType type;
    int userid;
    int roomid;
};//发给服务器 服务器会同步房间成员信息

//房间 为了避免0 出现歧义 房间号是0 还是没有初始化 把0让出来 1-120

#define DEF_JOIN_ROOM_RS    (_DEF_PACK_BASE + 7 )
enum ENUM_ROOM_STATUS{ _host, _player, _watcher };//房主 玩家 观战者
//加入房间回复
struct STRU_JOIN_ROOM_RS{
    //解决这是什么包 谁 哪个房间 叫什么名字
    STRU_JOIN_ROOM_RS():type( DEF_JOIN_ROOM_RS ),
        userid(0),
        roomid(0),
        status(_host),
        result(1){

    }
    PackType type;
    int userid;
    int roomid;
    int status;
    int result;// 0 fail 1 success
};

#define DEF_ROOM_MEMBER     ( _DEF_PACK_BASE + 8 )
//房间成员
struct STRU_ROOM_MEMBER{
    //解决这是什么包 谁 哪个房间 叫什么名字
    STRU_ROOM_MEMBER():type( DEF_ROOM_MEMBER ),
        userid(0), status(_player){
        memset( name, 0, sizeof(name) );
    }
    PackType type;
    int userid;
    //加上身份
    int status;
    char name[_MAX_SIZE];
};

#define DEF_LEAVE_ROOM_RQ   ( _DEF_PACK_BASE + 9 )
//退出房间
struct STRU_LEAVE_ROOM_RQ{
    //解决这是什么包 谁 退出了房间
    STRU_LEAVE_ROOM_RQ():type( DEF_LEAVE_ROOM_RQ ),
        userid(0),status(_player),roomid(0){

    }
    PackType type;
    int userid;
    int status;
    int roomid;
};//会被转发 如果自己不是房主 房主退出 自己也跟着退出

```

clogic.cpp

```cpp
#include "clogic.h"

void CLogic::setNetPackMap()
{
    NetPackMap(_DEF_PACK_REGISTER_RQ)    = &CLogic::RegisterRq;
    NetPackMap(_DEF_PACK_LOGIN_RQ)       = &CLogic::LoginRq;
    NetPackMap(DEF_PACK_JOIN_ZONE)       = &CLogic::JoinZoneRq;
    NetPackMap(DEF_PACK_LEAVE_ZONE)      = &CLogic::LeaveZoneRq;
    NetPackMap(DEF_JOIN_ROOM_RQ)         = &CLogic::JoinRoomRq;
    NetPackMap(DEF_LEAVE_ROOM_RQ)        = &CLogic::LeaveRoomRq;
}

#define _DEF_COUT_FUNC_    cout << "clientfd:"<< clientfd << __func__ << endl;

//注册
void CLogic::RegisterRq(sock_fd clientfd,char* szbuf,int nlen)
{
    //cout << "clientfd:"<< clientfd << __func__ << endl;
    _DEF_COUT_FUNC_
    //解析数据包 获取tel pasword name
    STRU_REGISTER_RQ * rq = (STRU_REGISTER_RQ *)szbuf;
    STRU_REGISTER_RS  rs;
    //根据tel 查数据库 看有没有
    list<string> lstRes;
    char sqlstr[1000] = "";
    sprintf( sqlstr, "select tel form t_user where tel = '%s';", rq->tel);
    m_sql->SelectMysql( sqlstr, 1, lstRes );
    //有返回结果 没有看接下来的昵称 有返回结果 没有注册成功
    //更新数据库 写表
    if( lstRes.size() > 0){
        rs.result = tel_is_exist;
    }
    else{
        lstRes.clear();
        sprintf( sqlstr, "select name form t_user where name = '%s';", rq->name);
        m_sql->SelectMysql( sqlstr, 1, lstRes );
        if( lstRes.size() > 0 ){
            rs.result = name_is_exist;
        }
        else{
            rs.result = register_success;
            sprintf( sqlstr, "insert into t_user ( tel, password, name ) value('%s','%s','%s');",
                     rq->tel, rq->password, rq->name );
            bool res = m_sql->UpdataMysql( sqlstr );
            if(!res){
                printf("update failed: %s\n", sqlstr);
            }
        }
    }
    //返回结果
    SendData( clientfd, (char * )&rs, sizeof(rs) );
}

//登录
void CLogic::LoginRq(sock_fd clientfd ,char* szbuf,int nlen)
{
    //cout << "clientfd:"<< clientfd << __func__ << endl;
    _DEF_COUT_FUNC_

    //STRU_LOGIN_RS rs;
    //rs.result = password_error;
    //SendData( clientfd , (char*)&rs , sizeof rs );

    //拆包 获取tel password
    STRU_LOGIN_RQ * rq = (STRU_LOGIN_RQ *)szbuf;
    STRU_LOGIN_RS rs;
    //根据tel查询id password name
    char sqlstr[1000] = "";
    list<string> lstRes;
    sprintf( sqlstr, "select id, password, name from t_user where tel = '%s';"
             , rq->tel );
    m_sql->SelectMysql( sqlstr, 3, lstRes );
    //没有返回结果 有看密码是否一致 不一致返回结果
    if( lstRes.size()==0 ){
        rs.result = user_not_exist;
    }
    else{
        int id = atoi( lstRes.front().c_str() );
        lstRes.pop_front();
        string strPasswd = lstRes.front();
        lstRes.pop_front();
        string strName = lstRes.front();
        lstRes.pop_front();
        if( strcmp( rq->password, strPasswd.c_str()) !=0 ){
            rs.result = password_error;
        }
        else{
            rs.result = login_success;
            UserInfo *info = nullptr;
            //如果之前有用户信息 强制下线 回收
            if( m_mapFdToUserInfo.find( id,info ) ){
                //强直下线

                //回收
                m_mapFdToUserInfo.erase(id);
                delete info;
            }
            //一致，把套结字和id捆绑在一起————非常重要
            //保存用户信息
            info = new UserInfo;
            info->m_id = id;
            info->m_sockfd = clientfd;
            strcpy( info->m_userName, strName.c_str() );

            strcpy( rs.name, strName.c_str() );
            rs.userid = id;
            //id 捆绑套结字
            m_mapFdToUserInfo.insert( id, info );
            //返回结果id name result
            SendData( clientfd, (char *)&rs, sizeof(rs) );
            //好友列表

            //离线信息

            //推送信息

            return;
        }
    }
    SendData( clientfd, (char *)&rs, sizeof(rs) );
}
/*
 * time 2025.12.26
 * add JoinZoneRq
 */
void CLogic::JoinZoneRq(sock_fd clientfd, char *szbuf, int nlen)
{
    printf("clientfd:%d JoinZoneRq\n", clientfd);
    //拆包
    STRU_JOIN_ZONE *rq = (STRU_JOIN_ZONE *)szbuf;
    UserInfo* info = nullptr;
    if( !m_mapFdToUserInfo.find( rq->userid, info) ){
        return;
    }
    info->m_zoneid = rq->zoneid;
}

void CLogic::LeaveZoneRq(sock_fd clientfd, char *szbuf, int nlen)
{
    printf("clientfd:%d LeaveZoneRq\n", clientfd);
    STRU_LEAVE_ZONE *rq = ( STRU_LEAVE_ZONE * )szbuf;
    UserInfo* info = nullptr;
    if( !m_mapFdToUserInfo.find( rq->userid, info) ){
        return;
    }
    info->m_zoneid = 0;
}
/*
 * time 2025.12.27
 * joinroomrq
 * 加入房间请求 加入时可能有多个线程同时有客户端请求 房间列表应该加锁处理
*/
void CLogic::JoinRoomRq(sock_fd clientfd, char *szbuf, int nlen)
{
    printf("clientfd:%d JoinRoomRq\n", clientfd);
    //拆包
    STRU_JOIN_ROOM_RQ * rq = (STRU_JOIN_ROOM_RQ *)szbuf;
    STRU_JOIN_ROOM_RS rs;
    list<int> tmplist;
    pthread_mutex_lock( &m_roomListMutex );
    list<int> & userlst = m_roomUserList[rq->roomid];
    //首先 0-120数组 看房间里人数
    //0 加入房间的就是 房主 返回
    //1 玩家 需要同步信息 玩家给加入的人发 加入的人给玩家发
    //2 加入失败
    switch( userlst.size() ){
    case 0:
        rs.result = 1;
        rs.roomid = rq->roomid;
        rs.status = _host;
        rs.userid = rq->userid;
        userlst.push_back( rq->userid );
        break;
    case 1:
        rs.result = 1;
        rs.roomid = rq->roomid;
        rs.status = _player;
        rs.userid = rq->userid;
        userlst.push_back( rq->userid );
        break;
    case 2:
        rs.result = 0;
        break;
    default:
        rs.result = 0;
        break;
    }
    tmplist = userlst;
    pthread_mutex_unlock( &m_roomListMutex );

    SendData( clientfd, (char*)&rs, sizeof(rs) );
    //player to host and host to player
    //size 为1 自己给自己发
    if( tmplist.size() > 0 ){
        //这个loginid名字起的不好
        int joinid = rq->userid;
        //根据id拿到用户信息
        UserInfo* joinInfo = nullptr;
        if( !m_mapFdToUserInfo.find(joinid, joinInfo))return;
        //写成员信息的请求
        STRU_ROOM_MEMBER joinrq;
        joinrq.userid = joinid;
        joinrq.status = rs.status;
        strcpy( joinrq.name, joinInfo->m_userName );
        bool flag = false;
        for( auto ite = tmplist.begin(); ite != tmplist.end(); ++ite){
            int status = _player;
            if(!flag){
                status = _host;
                flag = false;
            }
            int roomMemid = *ite;
            if(roomMemid != joinid){
                //根据id拿到用户信息
                UserInfo* MemInfo = nullptr;
                if( !m_mapFdToUserInfo.find(roomMemid, MemInfo))continue;
                //写成员信息的请求
                STRU_ROOM_MEMBER Memrq;
                Memrq.userid = roomMemid;
                Memrq.status = status;
                strcpy( Memrq.name, MemInfo->m_userName );
                //互相发送
                SendData( joinInfo->m_sockfd, (char *)&Memrq, sizeof(Memrq) );
                SendData( MemInfo->m_sockfd, (char *)&joinrq, sizeof(joinrq) );
            }
            else{
                //自己给自己发一条
                SendData( joinInfo->m_sockfd, (char *)&joinrq, sizeof(joinrq) );
            }
        }
    }
}
/*
 * time 2025.12.29
 * 离开房间
 */
void CLogic::LeaveRoomRq(sock_fd clientfd, char *szbuf, int nlen)
{
    printf("clientfd:%d LeaveRoomRq\n", clientfd);
    //拆包
    STRU_LEAVE_ROOM_RQ *rq = (STRU_LEAVE_ROOM_RQ *)szbuf;
    //谁 什么身份 离开哪个房间
    //rq->roomid;
    //rq->status;
    int leaveid = rq->userid;
    //根据身份不同 房主 player 操作 list 房间信息
    list<int>& lst = m_roomUserList[rq->roomid];
    //给除了这个离开的人之外的所有人发离开信息
    for(auto ite = lst.begin();ite!=lst.end();++ite){
        int memid = *ite;
        if( leaveid == memid ){
            UserInfo* memInfo = nullptr;
            if( !m_mapFdToUserInfo.find( memid, memInfo ) ) continue;
            SendData( memInfo->m_sockfd, szbuf, nlen );
            printf("clientfd:%d LeaveRoomRq SEND-FINISH\n", clientfd);
        }
        printf("clientfd:%d LeaveRoomRq FOR-CIRCLE finish\n", clientfd);
    }
    pthread_mutex_lock( &m_roomListMutex );
    if( rq->status == _host ){
        printf("clientfd:%d LeaveRoomRq HOST——LEAVE finish\n", clientfd);
        lst.clear();
    }
    else if( rq->status == _player ){
        //找到离开的人 清掉
        for(auto ite = lst.begin();ite!=lst.end();++ite){
            if( leaveid == *ite ){
                ite = lst.erase(ite);
                printf("clientfd:%d LeaveRoomRq FOR-CIRCLE PLAYER——LEAVE finish\n", clientfd);
                break;
            }
        }
    }
    pthread_mutex_unlock( &m_roomListMutex );

    printf("clientfd:%d LeaveRoomRq FINISH \n", clientfd);
    //当前人的离开要发给房间里的所有人
    /* 个人想法:
     * 当前人的离开要发给房间里的所有人
     * 个人感觉怪怪的 只发给房间里的人就够了么？
     * 如果有第三个人在，他也应该能看到房间状态的实时变化，
     * 如果第三个人是刚进入五子棋大厅，他可以获得房间的最新状况，
     * 如果是在五子棋大厅挂机，他应该是看不到其他房间的玩家流动的吧
     * 如果这样的话，
     * 个人的结构体里面应该加上是否在线的信息，将任何人的加入退出情况发给所有人
     * 如果可以的话，房间信息不应该用list存储，我的话应该会用拉链法的哈希表
     * 理由如下：1、房间的个数不需要频繁的变化，数组完全能够；
     * 2、房间内部是链表存储，加入退出很容易，头节点指向的就是房主，可以剩下一个int变量的空间？
     * ---------------------分割线------------------------
     * 他确实解决了 就是方法我没想到，单独写了一个协议，每隔几秒更新一次。。。
     */

}

```

time 2025.12.29 ckernel slot_dealRoomMemberRq fix bug: change rq->status into rq->userid

time 2025.12.29 CLogic LeaveRoomRq fix bug: send data to leaveid... why send data to myself?



定位结论（简短）
- 主要有两处严重逻辑错误：
  1. 客户端在接收房间成员同步包时，给 setPlayerInfo 传错参数（传了 status 而不是 userid），导致本地成员 id 列表 m_roomUserList 存错值，后续 playerLeave 无法找到并删除真实 id，从而界面不被重置，同步失败。
  2. 服务端在处理离开房间（LeaveRoomRq）时，发送通知的逻辑错误 —— 它把离开的消息发回给“离开的人自己”而不是发给房间中“其他成员”。也就是说服务器没有把离开事件广播给留下的成员，导致其他客户端没有收到离开通知无法更新 UI。
- 另外发现 JoinRoomRq 内给成员设置 status 的逻辑也有 bug（flag 没被置为 true），会把每个成员错误地标记为 host，从而导致别人接收的 status 信息不正确，这会进一步混淆客户端状态。

下面给出修复建议与具体补丁（已修复以上三处错误），并说明为什么这样改能修复问题。

要点说明（为什么修复）
- 修复 1（客户端）：把 setPlayerInfo 的第一个参数改为 rq->userid（真实用户 id），保证 m_roomUserList 中记录真实 id，playerLeave() 才能按 id 删除并更新对应槽位 UI。
- 修复 2（服务端）：在离开时把离开通知发给房间内除了离开者以外的成员（广播给剩下的人），这样剩下的客户端才会收到离开事件并更新 UI。
- 修复 3（服务端 JoinRoomRq）：修正 first-flag 逻辑，确保只有第一个成员被标记为 host，后续成员是 player，这样发给客户端的 status 信息才是正确的（防止混乱）。

补丁：修改的文件和代码如下。

```cpp name=ckernel.cpp
// （只贴修改的函数片段，其他保持不变）
```

修改：ckernel.cpp 的 slot_dealRoomMemberRq（修复 setPlayerInfo 参数）

```cpp name=ckernel.cpp
void CKernel::slot_dealRoomMemberRq(unsigned int lSendIP, char *buf, int nlen)
{
    //拆包 别人给你发 自己给自己发
    STRU_ROOM_MEMBER *rq = (STRU_ROOM_MEMBER *)buf;
    //设计的时候要加个身份——加身份了
    if( rq->status == _host ){
        m_roomDialog->setHostInfo( rq->userid,
                                  QString::fromStdString(rq->name));
    }
    if( rq->status == _player ){
        // 修复：这里应传入 userid，而不是 status
        m_roomDialog->setPlayerInfo( rq->userid,
                                  QString::fromStdString(rq->name));
    }

    m_roomDialog->setUserStatus( m_isHost?_host:_player);
}
```

解释：原代码把 rq->status（即 _player）当作 id 传给 setPlayerInfo，导致本地 m_roomUserList.push_back(...) 存入错误值（1），playerLeave 无法用真实 userid 删除对应槽位，从而 UI 不会重置。

---

```cpp name=clogic.cpp
// （只贴修改的函数片段，其他保持不变）
```

修改：clogic.cpp 的 LeaveRoomRq（修复通知广播逻辑）和 JoinRoomRq（修复 host 标记逻辑）

```cpp name=clogic.cpp
void CLogic::JoinRoomRq(sock_fd clientfd, char *szbuf, int nlen)
{
    printf("clientfd:%d JoinRoomRq\n", clientfd);
    //拆包
    STRU_JOIN_ROOM_RQ * rq = (STRU_JOIN_ROOM_RQ *)szbuf;
    STRU_JOIN_ROOM_RS rs;
    list<int> tmplist;
    pthread_mutex_lock( &m_roomListMutex );
    list<int> & userlst = m_roomUserList[rq->roomid];
    //首先 0-120数组 看房间里人数
    switch( userlst.size() ){
    case 0:
        rs.result = 1;
        rs.roomid = rq->roomid;
        rs.status = _host;
        rs.userid = rq->userid;
        userlst.push_back( rq->userid );
        break;
    case 1:
        rs.result = 1;
        rs.roomid = rq->roomid;
        rs.status = _player;
        rs.userid = rq->userid;
        userlst.push_back( rq->userid );
        break;
    case 2:
        rs.result = 0;
        break;
    default:
        rs.result = 0;
        break;
    }
    tmplist = userlst;
    pthread_mutex_unlock( &m_roomListMutex );

    SendData( clientfd, (char*)&rs, sizeof(rs) );
    //player to host and host to player
    if( tmplist.size() > 0 ){
        int joinid = rq->userid;
        UserInfo* joinInfo = nullptr;
        if( !m_mapFdToUserInfo.find(joinid, joinInfo)) return;
        //写成员信息的请求
        STRU_ROOM_MEMBER joinrq;
        joinrq.userid = joinid;
        joinrq.status = rs.status;
        strcpy( joinrq.name, joinInfo->m_userName );
        // 修复：确保第一个元素被标记为 host，之后为 player
        bool first = true;
        for( auto ite = tmplist.begin(); ite != tmplist.end(); ++ite){
            int status = _player;
            if( first ){
                status = _host;
                first = false; // <- 之前代码遗漏了这句，导致每个都被当做 host
            }
            int roomMemid = *ite;
            if(roomMemid != joinid){
                UserInfo* MemInfo = nullptr;
                if( !m_mapFdToUserInfo.find(roomMemid, MemInfo)) continue;
                STRU_ROOM_MEMBER Memrq;
                Memrq.userid = roomMemid;
                Memrq.status = status;
                strcpy( Memrq.name, MemInfo->m_userName );
                //互相发送：把已有成员信息发给 joiner，把 joiner 信息发给已有成员
                SendData( joinInfo->m_sockfd, (char *)&Memrq, sizeof(Memrq) );
                SendData( MemInfo->m_sockfd, (char *)&joinrq, sizeof(joinrq) );
            }
            else{
                //自己给自己发一条
                SendData( joinInfo->m_sockfd, (char *)&joinrq, sizeof(joinrq) );
            }
        }
    }
}

void CLogic::LeaveRoomRq(sock_fd clientfd, char *szbuf, int nlen)
{
    printf("clientfd:%d LeaveRoomRq\n", clientfd);
    //拆包
    STRU_LEAVE_ROOM_RQ *rq = (STRU_LEAVE_ROOM_RQ *)szbuf;
    int leaveid = rq->userid;
    list<int>& lst = m_roomUserList[rq->roomid];
    // 给除了离开的人之外的所有人发离开信息（广播给剩余成员）
    for(auto ite = lst.begin(); ite != lst.end(); ++ite){
        int memid = *ite;
        if( memid == leaveid ) continue; // 跳过离开者
        UserInfo* memInfo = nullptr;
        if( !m_mapFdToUserInfo.find( memid, memInfo ) ) continue;
        SendData( memInfo->m_sockfd, szbuf, nlen );
    }

    pthread_mutex_lock( &m_roomListMutex );
    if( rq->status == _host ){
        printf("clientfd:%d LeaveRoomRq HOST——LEAVE finish\n", clientfd);
        lst.clear();
    }
    else if( rq->status == _player ){
        //找到离开的人 清掉
        for(auto ite = lst.begin();ite!=lst.end();++ite){
            if( leaveid == *ite ){
                ite = lst.erase(ite);
                printf("clientfd:%d LeaveRoomRq FOR-CIRCLE PLAYER——LEAVE finish\n", clientfd);
                break;
            }
        }
    }
    pthread_mutex_unlock( &m_roomListMutex );

    printf("clientfd:%d LeaveRoomRq FINISH \n", clientfd);
}
```

解释：
- LeaveRoomRq 的旧实现只会把离开包发给等于 leaveid 的 memid（也就是发回给离开者本人），而不会通知剩余成员。所以剩下客户端不知道有人离开，无法重置界面。上述改为“遍历列表，给 memid != leaveid 的每位成员发送离开包”，即可通知所有剩余成员更新 UI。
- JoinRoomRq 中 first 标志必须在第一次使用后置为 true -> false，否则 every iteration 会被标记为 host。

---

```cpp name=roomdialog.cpp
// （只贴修改的函数片段，其他保持不变）
```

修改：RoomDialog::playerLeave（避免总是清空 player2，而是按 id 找到对应槽位清空）

```cpp name=roomdialog.cpp
void RoomDialog::playerLeave(int id)
{
    // 在 m_roomUserList 中找到 id，确定它是第几个槽位（0 为 host, 1 为 player），并清空对应 UI
    int pos = 0;
    bool found = false;
    for(auto ite = m_roomUserList.begin(); ite != m_roomUserList.end(); ++ite, ++pos){
        if(id == *ite){
            found = true;
            // 清空对应槽位 UI
            if(pos == 0){
                ui->lb_player1_name->setText("等待加入");
                ui->lb_icon_player1->setPixmap(QPixmap(":/Resource2/icon/slotwait.jpg"));
            }
            else if(pos == 1){
                ui->lb_player2_name->setText("等待加入");
                ui->lb_icon_player2->setPixmap(QPixmap(":/Resource2/icon/slotwait.jpg"));
            }
            // 从列表中删除
            ite = m_roomUserList.erase(ite);
            break;
        }
    }
    // 若没找到（数据不一致）也做一个兜底：清空 player2
    if(!found){
        ui->lb_player2_name->setText("等待加入");
        ui->lb_icon_player2->setPixmap(QPixmap(":/Resource2/icon/slotwait.jpg"));
        // 尝试清除任何匹配的 id
        for(auto ite = m_roomUserList.begin(); ite != m_roomUserList.end(); ){
            if(id == *ite){
                ite = m_roomUserList.erase(ite);
            } else ++ite;
        }
    }
}
```

解释：原实现无论哪个玩家离开，总是清空 player2，这在 host/玩家位顺序变动时会导致错误。新实现根据 m_roomUserList 内 id 的位置确定清理哪个槽位（0->player1/host，1->player2）。

---

验证建议（如何复现并确认修复）
1. 部署上述修改后的服务端和客户端。
2. 用三个客户端复现：
   - C 进入房间（作为观众），A、B 先后加入成为房主/玩家；
   - 让 A、B 依次离开（或同时离开），观察 C 是否收到离开通知并看到对应头像/名字恢复“等待加入”。
3. 在服务端日志添加调试输出（SendData 前后打印）以确认通知确实发出并且发往正确的 sockfd；在客户端 slot_dealLeaveRoomRq 和 slot_dealRoomMemberRq 打印接收到的包内容以确认 payload 正确（userid / status / name）。

补充注意事项
- 确保客户端和服务端关于 slot（host/player）顺序的约定一致：服务端在加入时应该保证先 push_back 的是 host，第二个是 player，客户端按照相同顺序更新 UI。
- 如果未来想更健壮，建议服务端增加一个专门的“房间状态广播”协议（把整个房间的 slot 数组发送给房内或所有相关客户端），客户端收到后完全覆盖本地显示（而不是靠增量通知），这能避免并发、顺序带来的状态不一致。
- 在多线程环境下房间列表操作应靠 mutex 保证一致性（代码已有 mutex，但发送通知/读取状态和随后的修改要注意先后顺序，最好在拿到锁后构造要广播的副本，然后释放锁再发送）。

如果你愿意，我可以：
- 把上面补丁整理成完整的 diff/patch（或 pull request 风格）供直接应用；
- 或者帮你在当前仓库中定位其它可能关联的同步问题（例如断开连接时没有触发 LeaveRoomRq）。

