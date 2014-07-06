---
layout: post
title:  ����HttpClient������֮һ��֧�ִ����Socket��װ
---

������ع���ӡ�����粿�ֵĴ��롪��ԭ�������ʱд��ʱ��̫�ż����ܶණ����û�кúõķ�װ�����������þ����ˡ���Ų��Ebox��Ϊ�˴�˵�е�ƽ̨�޹أ���FileStream��Socket֮������˴���JUCE�ķ�װ�������쳣���ģ�����������ԭ��Ŀǰ�����У����ǿ�ʼ�ع��ⲿ�ֵĴ���(ʵ�������Լ���ͷд�𡭡�)��ĿǰҲ���˸�����ˣ����ǿ���дд�ܽ᣺������HttpClient����������

��˵���������£�������������������HttpClient�������ǿ�֧�ִ����Socket��װ�ࡪ�������Լ�дHttpClient������ʹ��WinInet֮��Ŀ�ܴ󲿷�ԭ�����WinInetֻ֧��Http�������֧��Socks�Ĵ�������ֻ���Լ����ַ�����ʳ�ˡ�(��˵�����΢��ĺܶණ����������ֻ֧��HTTP��������C#�бȽ�������ProxySocket ���Ǳ���д��)��C++����ò��Ҳû��֧�ִ����HTTP�⣬����C�����и�curl������Լ����÷�װHttp���ǿ��������á�

�Թ�������˵ProxySocket�ķ�װ��������˵�����������Ϊ���֣�http,ftp��socks������͸������(���繫˾�ķ�ǽ���������͸������)�ͷ�͸������֮�֡���������ֻ����http��socks�����ֱȽϳ����ķ�͸����������ftp����ò��ֻ�ܴ������������ˣ�����������ʲô�������ṩ�����Ĵ���������ͨ���������������ʱ������ͨѶ���̷�Ϊ�������֣������ʹ���ͨѶ�������Ŀ�ĵ�ַͨѶ������һ���ͻ�����Ҫ���ĵľ�ֻ�У��ʹ�����������һ�ζԴ���Э��Э�����ֹ��̣�����֮��Ϳ��԰Ѵ������������Ŀ���ַ�ˡ�

##HTTP����

���ͻ���������һ��HTTP�����������ͨ�����������󣬴������������������ǣ�������Ŀ���ַ�����ӣ��������󣬽��ܷ��������������ؿͻ��ˡ�Ϊ��ʵ����һ�㣬��HTTPЭ���й涨����ôһ�����ⷽ����CONNECT�����ͻ��˺�HTTP������������Ӻ�ֻ��Ҫ�������¸�ʽ��HTTP���󼴿ɣ�

CONNECT <destanation_address> : <destanition_port> <http_version><CR><LF>          
Host: <destanation_address> : <destanition_port><CR><LF> 
<optional_header_line><CR><LF> 
<optional_header_line><CR><LF> 
���� 
<CR><LF>�����������Ҫ��֤�û�����������Ҫ��"�û���������"����Base64��������룺
CONNECT <destanation_address> : <destanition_port> <http_version><CR><LF>          
Host: <destanation_address> : <destanition_port><CR><LF> 
Authorization: Basic <base64�����֤�ֶ�><CR><LF> 
Proxy-Authorization: <Basic base64�����֤�ֶ�><CR><LF> 
<optional_header_line><CR><LF> 
���� 
<CR><LF>


{% highlight C++ %}
char buff[kmax_file_buffer_size] = {0};
if (!_proxy_config._username.empty())
{
	std::string auth = _proxy_config._username + ":" + _proxy_config._password;
	std::string base64_encode_auth;
	Util::base64Encode(auth,base64_encode_auth);
	sprintf_s(buff,kmax_file_buffer_size,"CONNECT %s:%d HTTP/1.1rnHost: %s:%drnAuthorization: Basic %srnProxy-Authorization: Basic %srnrn",
	_host_name.c_str(),_port_number,_host_name.c_str(),_port_number,base64_encode_auth.c_str(),base64_encode_auth.c_str());
}
else
{
	sprintf_s(buff,kmax_file_buffer_size,"CONNECT %s:%d HTTP/1.1rnHost: %s:%drnrn",
	_host_name.c_str(),_port_number,_host_name.c_str(),_port_number);
}
//����HTTP������������
bool send_connect_request = _socket.writeAll(buff,strlen(buff));
if (!send_connect_request)
{
	return false;
}
//���HTTP����ظ�
int ret = _socket.read(buff,sizeof(buff));
if (ret <= 0)
{
	return false;
}
buff[ret] = '';
Util::makeLower(buff,strlen(buff));
return strstr(buff, "200 connection established") != 0;
{% endhighlight %}


�������������յ���ȷ��������ʾ�������ӳɹ���(һ�����code = 200)����ֵ��˵����һ���ǣ���������ֻ�Ƕ�Basic������֤��ʽ���˴����������Ĵ������ʽ�Ǻܲ���ȫ�ġ���Ȼ����һ����֤��ʽ��NTLM����Զ��ԱȽϸ��ӣ���׸����(ʵ������û��ȥ����)
##Socks4����
Socks4�Ĵ��������������Http����Ҫ�򵥲��٣�����Socks4�ǲ�֧���û���֤�ġ�����Socks4������Ͱ���5���ֶζ��ѣ�

* �ֶ�һ��Socks�汾�ţ���0��04��ռһ���ֽڡ�
* �ֶζ��������룬ռһ���ֽڣ�����0��01��TCP/IP���ӣ���0��02���˿ڰ󶨡�
* �ֶ����������ֽ���˿ڣ�ռ�����ֽڡ�
* �ֶ��ģ������ֽ���IP��ַ��ռ�ĸ��ֽڡ�
* �ֶ��壺�û�ID�ֶΣ��ɱ䣬��null(0)��β��
�����������صķ�������Ӽ򵥣�һ������4���ֶΣ�
* �ֶ�һ��һ�����ֽ�
* �ֶζ���һ���ֽڣ���ʾ����״̬�룬����0x5A(��90)��ʾ���󱻽��ܡ�
* �ֶ����������ֽڣ��ɱ�����
* �ֶ��ģ��ĸ��ֽڣ��ɱ�����

���Կ���ʵ��������Sock4������ͷ��������쳣�򵥣�SOCK4�ķ�������ֻ��һ���ֽ���������ģ������ɾͿ��Ը㶨��
{% highlight C++ %}
//Socks4û���û�������֤
struct Sock4Reqeust
{
	char VN;
	char CD;
	unsigned short port;
	unsigned long ip_address;
	char other[256]; // �䳤
} sock4_request;
struct Sock4Reply
{
	char VN;
	char CD;
	unsigned short port;
	unsigned long ip_address;
} sock4_reply;
sock4_request.VN = 0x04; // VN��SOCK�汾��Ӧ����4��
sock4_request.CD = 0x01; // CD��SOCK�������룬1��ʾCONNECT����2��ʾBIND����
sock4_request.port= ntohs(_port_number);
sock4_request.ip_address = SocketHelper::getIntAddress(_host_name.c_str());
sock4_request.other[0] = '';
if (sock4_request.ip_address == INADDR_NONE)
return false;
//����SOCKS4��������
bool send_sock4_requst =  _socket.writeAll((char*)&sock4_request,9);          
if (!send_sock4_requst)
{
	return false;
}
//���Socks4����Ļظ�
int ret = _socket.read((char *)&sock4_reply, sizeof(sock4_reply));
if (ret <= 0)
{          
	return false;
}
/*
CD�Ǵ���������𸴣��м��ֿ��ܣ�
90������õ�����
91�����󱻾ܾ���ʧ�ܣ�
92������SOCKS�������޷����ӵ��ͻ��˵�identd��һ����֤��ݵĽ��̣������󱻾ܾ���
93�����ڿͻ��˳�����identd������û���ݲ�ͬ�����ӱ��ܾ���
*/
return sock4_reply.CD == 90;
{% endhighlight %}
ֵ��ע��������ڴ��������ýṹ���ʾ��Ӧ���ֶμ��ϣ�Ҫ���ǵ��ֽڶ���Խṹ���ֶ��ŷŵ�Ӱ����Ҫָ��**#pragma pack(1)**��ǿ����һ���ֽڶ��롣Socks4����һ�ֱ��ֽ���Socks4a������Ͳ������ˣ����Բο����

##Socks5����

Socks5��Socks4��һ�������汾�������˺ܶ�Socks4��֧�ֵ����ԣ������IPv6��֧�֡�һ��������Socks5��������Э����Է�Ϊ����������裺

* �ͻ��˷����������󣬲���Ϊ��֧�ֵ���֤�����б� 
* ������ѡ��һ����֤��ʽ������(����޿�֧�ֵ���֤��ʽֱ�ӷ���ʧ��)
* ������ѡ��֤��ʽ���ͻ��˺ͷ���˽�����֤��ʽ�ϵ�Э��(����ѡ�����û���/�������ַ�ʽ������֤���ͻ��˾���Ҫ�����û��������ȥ��Ȼ�������������֤�����ؽ��)
* �ͻ��˷���һ��������Socks4������
* ����˷���һ������Socks4�ķ���
��������������裬�ͻ��˺ͷ���˵����־�������ˣ�����������ֱ�ӽ������ݴ��伴�ɡ�
       Socks5��֧�ֵ���֤��ʽ������
       0��00������֤
       0��01��GSSAPI
       0��02���û���/����
       0��03-0x7F��IANAָ���ķ���
       0��80-0xFE����������
������������򵥵�ProxySocket��˵��ȫ����ֻ����0��00��0��02���ɣ��������ַ�ʽһ��������������Ҳû��̫���ʵ�ֱ�Ҫ��
�ͻ��˷���ĵ�һ�������ܹ����������ֶΣ�

* �ֶ�1:Socks�汾�ţ���0��05
* �ֶ�2:֧�ֵ���֤��ʽ����һ���ֽ�
* �ֶ�3:��֤�����б��䳤��һ���ֽڱ�ʾһ������

��������յ��������󣬷���һ����֤�����ķ�����һ���������ֶΣ�

* �ֶ�1��Socks�汾�ţ�һ���ֽڣ���0��05
* �ֶ�2��ѡ�����֤������һ���ֽڣ�����ڿͻ��˷�������֤�����б���û�з����֧�ֵķ������򷵻�0xFF

���ʱ��ͻ��˿��Ը��ݷ���˷���������֤����������֤��0��00��ֱ��������������һ�����͵��û���/������֤�������£�

* �ֶ�1���汾�ţ�һ���ֽڣ�����ָ��Ϊ0��01
* �ֶ�2���û������ȣ�һ���ֽ�
* �ֶ�3���û������䳤
* �ֶ�4�����볤�ȣ�һ���ֽ�
* �ֶ�5�����룬�䳤
����˷������µķ�����

* �ֶ�1���汾�ţ�һ���ֽ� (�����ʵ���Ժ���)
* �ֶ�2��״̬�룬һ���ֽڣ�0��00��ʾ�ɹ�������ֵ���ʾ��֤ʧ�ܣ�������Ҫ�Ͽ���
���Ժ�Ĺ����ͻ�����Sock4�����ˣ��ͻ��˷���һ����������

* �ֶ�1��Socks�汾�ţ�һ���ֽڣ���0��05
* �ֶ�2�������룬һ���ֽڣ�0��01��TCP/IP���ӣ�0��02��TCP/IP�˿ڰ󶨣�0��03������UDP�˿�
* �ֶ�3�������ֶΣ�һ���ֽڣ�����ָ��Ϊ0��00
* �ֶ�4����ַ���ͣ�һ���ֽڣ�0��01��IPv4��ַ��0��03��������0��04��IPv6��ַ
* �ֶ�5��Ŀ���ַ��4�ֽڱ�ʾ��IPv4��ַ������16�ֽڱ�ʾ��IPv6��ַ��������������Ǳ䳤��һ�ֽڱ�ʾ�������ȣ���������������
* �ֶ�6��������Ķ˿ںţ������ֽ�
�������ķ���Ϊ��

* �ֶ�1��SockЭ��汾��һ���ֽڣ�����Ϊ0��05
* �ֶ�2��״̬�룬0��00��ʾ����ɹ��������״̬��ɲο���Ӧ��RFC�ĵ�
* �ֶ�3�������ֶΣ�һ���ֽڣ������ƶ�Ϊ0��00
* �ֶ�4����ַ���ͣ�һ���ֽڣ�0��01��IPv4��ַ��0��03��������0��04��IPv6��ַ
* �ֶ�5��Ŀ���ַ��4�ֽڱ�ʾ��IPv4��ַ������16�ֽڱ�ʾ��IPv6��ַ��������������Ǳ䳤��һ�ֽڱ�ʾ�������ȣ���������������
* �ֶ�6��������Ķ˿ںţ������ֽ�
����һ��������Socks5����Э���������ˣ����Ǽ��ڴ���ƪ��̫���ˣ�����Ͳ����ˣ�������HttpClient������Ϻ���ͳһ�ϴ��롭��.��ʵ�����ǡ�ĳЩ�ط��Ĵ��뻹û����ˬ�����ռ����Ű���

##�ο�����
1.Wiki�й���Sock5�Ľ��ܣ�http://en.wikipedia.org/wiki/SOCKS
2.��Http Tunneling����http://www.codeproject.com/KB/IP/httptunneling.aspx
3.��CAsyncProxySocket �C CAsyncSocket derived class to connect through proxies����http://www.codeproject.com/KB/IP/casyncproxysocket.aspx




