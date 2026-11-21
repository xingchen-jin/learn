```lua
using System.Collections;

using System.Collections.Generic;

using System.Net;

using System.Net.Sockets;

using System.Text;

using UnityEngine;

  

public class Lesson14 : MonoBehaviour

{

    // Start is called before the first frame update

    void Start()

    {

        Socket socket = new Socket(AddressFamily.InterNetwork, SocketType.Dgram, ProtocolType.Udp);

        IPEndPoint iPEndPoint = new IPEndPoint(IPAddress.Parse("127.0.0.1"), 8086);

        socket.Bind(iPEndPoint);

        //指定要发送的字节数组和远程接收端的 IP地址和端口号

        string msg = "Hello, UDP!";

        IPEndPoint remoteIpEndPoint = new IPEndPoint(IPAddress.Parse("127.0.0.1"), 8087);

        socket.SendTo(Encoding.UTF8.GetBytes(msg), remoteIpEndPoint);

        //接收数据

        EndPoint remoteIpEndPoint2 = new IPEndPoint(IPAddress.Any, 0);

        byte[] buffer = new byte[1024];

        int length = socket.ReceiveFrom(buffer, ref remoteIpEndPoint2);

        print(string.Format("接收到 {0} 发来的消息: {1}" ,remoteIpEndPoint2.ToString(), Encoding.UTF8.GetString(buffer, 0, length)));

        //关闭

        socket.Shutdown(SocketShutdown.Both);

        socket.Close();

    }

  

    // Update is called once per frame

    void Update()

    {

    }

}
```