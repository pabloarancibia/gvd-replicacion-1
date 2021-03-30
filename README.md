# gvd-replicacion-1
TP1 Primera parte - Replicación con mongoDb

REPLICACIÓN EN MONGODB 

Levanto las 3 instancias para la Replica Set con los siguientes parámetros
mongod --replSet rs --dbpath /media/documentoslinux/iot/gvd/tp1/rs/0 --port 27018 --smallfiles --oplogSize 50
mongod --replSet rs --dbpath /media/documentoslinux/iot/gvd/tp1/rs/1 --port 27019 --smallfiles --oplogSize 50
mongod --replSet rs --dbpath /media/documentoslinux/iot/gvd/tp1/rs/2 --port 27020 --smallfiles --oplogSize 50

mongo --port 27018
	cfg = {
		_id:"rs",
		members:[
			{_id:0, host:"localhost:27018"},
			{_id:1, host:"localhost:27019"},
			{_id:2, host:"localhost:27020", arbiterOnly:true}
		]
	};
	rs.initiate(cfg)

----------------------------------------------------------
// me conecto a la shell del Primary

mongo --port 27018

//CREO DB

	use finanzas
----------------------------------------
// INSERTO REGISTROS 

	load("facts.js") //x4
---------------------------------------
// INDICO AL SECONDARY QUE DEBE COPIAR LOS DATOS DE PRIMARY
// VOY AL SHELL DEL SECONDARY Y EJECUTO EL COMANDO 

	rs.slaveOk()
	

// HAGO FAILOVER DEL PRIMARY PARA SIMULAR UNA CAIDA
//CTRL+C en la terminal del primary (no en la shell sino en la terminal donde está corriendo el rs)
// ahora podemos ver que en la shell del secondary si ahcemos enter vemos que se convirtio en primary

-------------------------------------
//CARGAMOS DATOS EN EL PRIMARY ACTUAL , ESTANDO CAIDO EL OTRO
// O TAMBIEN PODEMOS CREAR UNA DB Y AGREGAR UN DATO

	use finanzas
	db.facturas.insert({X:1})
	
// O
	use prueba
	db.registros.insert({X:1})

---------------------------------
//LEVANTO EL PRIMARY QUE ESTA CAIDO.
sudo mongod --replSet rs --dbpath /media/documentoslinux/iot/gvd/tp1/rs/0 --port 27018

// PUNTO 8 - AGREGAR NUEVO NODO
mongod --replSet rs --dbpath /media/documentoslinux/iot/gvd/tp1/rs/3 --port 27021

// VOY A LA SHELL DEL NUEVO RS Y LE ASIGNO SECONDARY OK
	mongo --port 27021
	rs.secondaryOk()

// EN LA SHELL DEL PRIMARY EJECUTO PARA DARLE DELAY
	rs.add({host:"localhost:27021", priority:0, slaveDelay: 120})
	
	
// COMPRUEBO:
	rs.conf()

// COMPRUEBO SINCRONIZACION EN 27021
	show.dbs

// ejecuto en las 3 shell un count
	db.facturas.count()	

// EJECUTO NUEVAMENTE facts.js En el primary
	load("/media/documentoslinux/iot/gvd/tp1/facts.js")
	
// Compruebo nuevamente con count en las 3 shell
//FIN

	

	
	
