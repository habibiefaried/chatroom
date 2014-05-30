
/**
 * Module dependencies.
 */

var express = require('express')
  , routes = require('./routes/index')
  , socketio = require('socket.io')
  , http = require('http')
  , path = require('path');

var app = express();

// all environments
app.set('port', process.env.PORT || 3000);
app.set('views', __dirname + '/views');
app.set('view engine', 'ejs');
app.use(express.favicon());
app.use(express.logger('dev'));
app.use(express.bodyParser());
app.use(express.methodOverride());
app.use(express.cookieParser('bermalamdisisterbesokmalam'));
app.use(express.session());
app.use(app.router);
app.use(require('less-middleware')({ src: __dirname + '/public' }));
app.use(express.static(path.join(__dirname, 'public')));

// development only
if ('development' == app.get('env')) {
  app.use(express.errorHandler());
}

app.get('/', routes.index);//route

var server = app.listen(app.get('port'), function(){
  console.log("Express server listening on port " + app.get('port'));
});

var io = socketio.listen(server);
//listening socket.io client in express port

/* Algoritma socket.io dimulai dari sini */

/* Deklarasi Struktur Data */
var user = {};
var room = {};
var userTop = 0;
//Jumlah user yang sedang online

var userJoined = {};
// ini adalah array of array

var userJoinedTop = {};
//ini merupakan array of jumlah user

/* End of Deklarasi struktur data */

io.sockets.on('connection', function (socket) {
	
socket.on("register",function(msg)
{
	if (user[msg.nama] == undefined) //jika belum ada yang pakai
	{
		user[msg.nama] = socket.id; //id user
		userTop++;
		socket.username = msg.nama; //session username
		socket.emit("registerReply",{message:"Selamat datang, "+msg.nama+"!"});

		//broadcast ke seluruh pengguna, semua room yang ada
		io.sockets.emit("ListRoom",{msg:room});
	}
	else //jika sudah ada
	{
		socket.emit("registerReply",{message:"0"});
	}	
});

socket.on("createRoom",function(msg)
{
	if (room[msg.room] == undefined)
	{
		room[msg.room] = socket.username; //creator
		socket.room = msg.room; //session room

		userJoined[msg.room] = {}; //inisialisasi
		userJoined[msg.room][socket.username] = socket.id; //user pertama
		userJoinedTop[msg.room] = 1;
		socket.emit("createReply",{message:"Room "+msg.room+" berhasil dibuat!"});

		socket.join(socket.room); //join ke roomnya sendiri

		//broadcast user yang join di roomnya sendiri
		io.sockets.in(msg.room).emit("ListUser",{msg:userJoined[msg.room]});
		
		//broadcast ke seluruh pengguna, semua room yang ada
		io.sockets.emit("ListRoom",{msg:room, baru:msg.room});
	}
	else
	{
		socket.emit("createReply",{message:"0"});
	}
});

socket.on("joinRoom",function(msg)
{
	if ((userJoinedTop[msg.nama] < 4) && (userJoinedTop[msg.nama] != undefined))
	//jika lebih kecil dari 4 dan tidak undefined
	{
		socket.room = msg.nama; //session room
		userJoined[socket.room][socket.username] = socket.id; //user selanjutnya
		userJoinedTop[socket.room]++; //increment jumlah user		
		socket.join(socket.room); //join ke room pilihan
		socket.emit("joinRoomReply",{msg:"Selamat datang di room "+socket.room});
		io.sockets.in(socket.room).emit("chat",
					{
						chat:"Username "+socket.username+" telah bergabung di room ini",
						name:"SERVER"
					}); 

		//broadcast user yang join di roomnya sendiri
		io.sockets.in(socket.room).emit("ListUser",{msg:userJoined[socket.room]});
		if (userJoinedTop[socket.room] >= 4) //jika sudah sama dengan atau lewat 4
		{
			delete room[socket.room];
		}
		//broadcast ke seluruh pengguna, semua room yang ada
		io.sockets.emit("ListRoom",{msg:room, baru:socket.room});
	}
	else
	{
		socket.emit("joinRoomReply",{msg:"0"});
		//broadcast ke seluruh pengguna, semua room yang ada
		io.sockets.emit("ListRoom",{msg:room, baru:socket.room});
	}
});

socket.on("msgChat",function(data)
{
	io.sockets.in(socket.room).emit("chat",
											{
											 chat:data.msg,
											 name:socket.username
											}); 
});

socket.on('disconnect', function(){
	if (userJoined[socket.room] != undefined)
	{
		if (userJoined[socket.room][socket.username])
			delete userJoined[socket.room][socket.username];
	}
	
	io.sockets.in(socket.room).emit("chat",
					{
						chat:"Username "+socket.username+" telah keluar dari room ini",
						name:"SERVER"
					}); 

	io.sockets.in(socket.room).emit("ListUser",{msg:userJoined[socket.room]});
});

});
