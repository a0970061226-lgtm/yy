import 'package:flutter/material.dart';
import 'package:socket_io_client/socket_io_client.dart' as io;

void main() => runApp(MaterialApp(home: AuthPage(), theme: ThemeData.dark(), debugShowCheckedModeBanner: false));

// --- 登入畫面 (已復原) ---
class AuthPage extends StatefulWidget {
  @override
  _AuthPageState createState() => _AuthPageState();
}

class _AuthPageState extends State<AuthPage> {
  final uC = TextEditingController(), pC = TextEditingController();
  late io.Socket socket;

  @override
  void initState() {
    super.initState();
    socket = io.io('http://192.168.0.45:3000', <String, dynamic>{'transports': ['websocket']});
    socket.on('auth', (d) {
      if (d['success']) {
        Navigator.pushReplacement(context, MaterialPageRoute(builder: (_) => AdminPage(socket: socket)));
      } else {
        ScaffoldMessenger.of(context).showSnackBar(SnackBar(content: Text("登入失敗")));
      }
    });
  }

  @override
  Widget build(BuildContext context) => Scaffold(body: Center(child: Column(mainAxisAlignment: MainAxisAlignment.center, children: [
    SizedBox(width: 250, child: TextField(controller: uC, decoration: InputDecoration(labelText: "帳號"))),
    SizedBox(width: 250, child: TextField(controller: pC, decoration: InputDecoration(labelText: "密碼"), obscureText: true)),
    ElevatedButton(onPressed: () => socket.emit('login', {'user': uC.text, 'password': pC.text}), child: Text("登入"))
  ])));
}

// --- 審核畫面 ---
class AdminPage extends StatefulWidget {
  final io.Socket socket;
  AdminPage({required this.socket});
  @override
  _AdminPageState createState() => _AdminPageState();
}

class _AdminPageState extends State<AdminPage> {
  List<String> pending = ['demo'];

  void _showApprovalFlow(int index) {
    showModalBottomSheet(context: context, builder: (_) => Column(mainAxisSize: MainAxisSize.min, children: [
      ListTile(title: Text("1. 總管理"), onTap: () => _apply(index, '總管理')),
      ListTile(title: Text("2. 永久會員"), onTap: () => _apply(index, '永久會員')),
      ListTile(title: Text("3. 一般會員"), onTap: () => _apply(index, '一般會員')),
      ListTile(title: Text("4. 體驗會員"), onTap: () => _apply(index, '體驗會員')),
    ]));
  }

  void _apply(int idx, String role) {
    widget.socket.emit('change_role', {'targetUser': pending[idx], 'newRole': role});
    Navigator.pop(context);
    setState(() => pending.removeAt(idx));
  }

  @override
  Widget build(BuildContext context) => Scaffold(
    appBar: AppBar(title: Text("待審核清單")),
    body: ListView.builder(itemCount: pending.length, itemBuilder: (_, i) => ListTile(
      leading: CircleAvatar(child: Text("${i + 1}")),
      title: Text("帳號: ${pending[i]}"),
      onTap: () => _showApprovalFlow(i),
    )),
  );
}
