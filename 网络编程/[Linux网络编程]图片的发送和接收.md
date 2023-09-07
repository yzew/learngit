> `记录项目中实现QT客户端发送图片，Linux服务器接收图片并保存本地的功能代码。`  
>  如果没有要求，可一次性把图片数据发送，不用分成多个包进行传输。

### 文章目录

  * 包的设计
  * QT客户端主要代码
  * linux客户端接收主要代码
  * 结果展示

# 包的设计

    
    
    typedef struct insert {
        int packet_seq;		//包序号
        int packet_sum;		//包总数
        char trans_id[32];	//包流水32
    }INSERT_T;
    
    typedef struct service	//业务层数据
    {
        int funcid;			//功能号
        char contex[900];	//业务数据
    }SERVICE_T;
    
    typedef struct bag {	//总包类型
        char bag_head[2];	//包头		    4字节
        int fd;				//fd            4字节
        INSERT_T insert_;	//接入层数据    40字节
        SERVICE_T service_;	//业务层数据    904字节
        int error_code;		//错误代码	    4字节
        char error_msg[32];	//错误信息	    32字节
        char bag_md5[32];	//校验码        32字节
        char bag_tail[2];	//包尾		    4字节
    }BAG_T;
    
    
    typedef struct req_picture
    {
        char picture_id[32];		//图片id（名字）
        char user_id[16];			//用户id
        char video_id[32];			//视频id（名字）
        char picture_time[32];		//拍摄时间
        char picture_locale[32];	//拍摄地点
        int status;					//自动拍照：0，手动拍照：1
        int len;					//图片大小
        unsigned char picture[700];			//存放视频
    }REQ_PICTURE_T;
    
    

# QT客户端主要代码

> 简单的说就是转换成dataArray的数据格式，根据实际包实际能存放的大小，每次发送固定长度的图片数据给服务器。注意最后一个包的处理。
    
    
    void picture::on_pushButton_2_clicked()
    {
        //设置图片id
        QDateTime current_date_time =QDateTime::currentDateTime();
        QString current_date =current_date_time.toString("yyyyMMddhhmmss");
        picture_id = QString("%1_%2.jpg").arg(current_date).arg(ui->lineEdit_3->text());
        ui->lineEdit_2->setText(picture_id);
    
        //打开一张图片
        QPixmap pix(file_path);
        QBuffer buffer;
    
        buffer.open(QIODevice::ReadWrite);
        pix.save(&buffer,"jpg");
    
        QByteArray dataArray;
        dataArray.append(buffer.data());
        quint32 pix_len = (quint32)dataArray.size();
        qDebug("image size:%d",pix_len);
    
        char buf[1024] = {0};       //发送到服务器的buf
        memset(buf,0,sizeof(buf));
        memset((void *)&my_picture,0,sizeof(REQ_VIDEO_T));
        memset((void *)&my_bag,0,sizeof(BAG_T));
    
        //设置包头包尾校验
        my_bag.bag_head[0] = 0x40;
        my_bag.bag_head[1] = 0x04;
        my_bag.bag_tail[0] = 0x50;
        my_bag.bag_tail[1] = 0x05;
        my_bag.error_code = 0;//错误代码
        strcpy(my_bag.error_msg,"");
        //业务层
        my_bag.fd = 0;
        my_bag.insert_.packet_sum = pix_len/512 + 1;//包总数ceil
        my_bag.service_.funcid = REQ_PICTURE_UPLOAD;//功能号
        QByteArray data;
        for(int i = 0;i <= pix_len/512;i++)
        {
            if(i != pix_len/512 + 1)
            {
                data = dataArray.left(512);
                my_picture.len = 512;
                dataArray.remove(0,512);
            }
            else 
            {
                data = dataArray.left(pix_len%512);
                my_picture.len = pix_len%512;
            }
    
            //接入层
            my_bag.insert_.packet_seq = i;//包序号
            get_serial_num(NULL,my_bag.insert_.trans_id);//流水号
            memcpy(my_picture.picture_id,ui->lineEdit_2->text().toStdString().c_str(),sizeof(my_picture.picture_id));
            memcpy(my_picture.user_id,ui->lineEdit_3->text().toStdString().c_str(),sizeof(my_picture.user_id));
            memcpy(my_picture.video_id,ui->lineEdit_4->text().toStdString().c_str(),sizeof(my_picture.video_id));
            memcpy(my_picture.picture_time,ui->timeEdit->text().toStdString().c_str(),sizeof(my_picture.video_id));
            memcpy(my_picture.picture_locale,ui->lineEdit_7->text().toStdString().c_str(),sizeof(my_picture.picture_locale));
            my_picture.status = 0;
            memcpy(my_picture.picture,data,my_picture.len);
            //将图片数据复制到业务数据
            memcpy(my_bag.service_.contex,&my_picture,sizeof(REQ_PICTURE_T));
    
            QCryptographicHash hash_buf(QCryptographicHash::Md5);
            hash_buf.addData((char*)&(my_bag.insert_),(sizeof(INSERT_T) + sizeof(SERVICE_T)));  // 添加数据到加密哈希值
            QByteArray result_buf = hash_buf.result();  // 返回最终的哈希值
            memcpy(my_bag.bag_md5, result_buf.data(),16);   //将得到的MD5考进去
    
            unsigned char temp[34];
            memset(temp,0,34);
            memcpy(temp, &my_bag.bag_md5,32);
            //将bag拷到buf
            memcpy(buf,&my_bag,sizeof(BAG_T));
            my_qTcpSocket->write(buf,sizeof(buf));
            QThread::msleep(2);
        }
    }
    

# linux客户端接收主要代码

> 简单的说就是打开文件写入数据，要么使用追加写入，要么利用lseek做偏移再写入。
    
    
    if(ret==1006)//表明是图片包
    	{
    		int fd = open(((REQ_PICTURE_T*)((BAG_T*)gdata_)->service_.contex)->picture_id, O_RDWR);
    		if (fd == -1) 
    		{
    			fd = open(((REQ_PICTURE_T*)((BAG_T*)gdata_)->service_.contex)->picture_id, O_CREAT | O_RDWR, 0777);
    		}
    		lseek(fd, ((BAG_T*)gdata_)->insert_.packet_seq * 512, SEEK_SET);
    		write(fd, ((REQ_PICTURE_T*)((BAG_T*)gdata_)->service_.contex)->picture, ((REQ_PICTURE_T*)((BAG_T*)gdata_)->service_.contex)->len);
    		close(fd);
    //主要代码就上面这段	
    		printf("图片保存成功\n");
    	}
    

# 结果展示

![在这里插入图片描述](https://img-
blog.csdnimg.cn/37ecc3bdbc7746118eb2b08b8443407b.png?x-oss-
process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAV2luZGFsb3Zl,size_19,color_FFFFFF,t_70,g_se,x_16)

![在这里插入图片描述](https://img-
blog.csdnimg.cn/a217473b9fd847a6882f05f936c023fb.png?x-oss-
process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAV2luZGFsb3Zl,size_9,color_FFFFFF,t_70,g_se,x_16)

> 如有错误，欢迎指出。

