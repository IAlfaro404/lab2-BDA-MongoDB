
## 1. Crear red personalizada y levantar contenedores con configuración de replicación
Para una configuración óptima, se va a usar Docker y Docker Compose, por lo que se debe tener instalado en el sistema. 

Para ello, se descarga la imagen de MongoDB:

```
docker pull mongo-latest
```

Se debe crear un Network:

```
docker network create mongodb-lab2-grupo2
```

Se levantar el contenedor mongo1 como primario con configuración de replicación y futuro primario del Replica Set. 

```
docker run -d -p 3001:27017 --name mongo1 --network mongodb-lab2-grupo2 mongo:latest mongod --replSet rs
```

Explicación:
- `d:` Ejecuta el contenedor en modo detached (segundo plano).
- `-name mongo1`: Asigna el nombre mongo1 al contenedor.
- `-network mongodb-lab2-grupo2`: Conecta el contenedor a la red que creamos.
- `-p 3001:27017`: Mapea el puerto 3001 del contenedor al puerto 27017 de la máquina local (para acceso desde fuera de Docker).
- `-replSet rs`: Inicializa el proceso mongod con el nombre de Replica Set rs.


Adicionalmente, se levanta los contenedores mongo2 y mongo3 como secundarios por lo que serán los nodos secundarios que se unirán al Replica Set.

```
docker run -p 3002:27017 --name mongo2 --network mongodb-lab2-grupo2 -d mongo:latest
    mongod --replSet rs

docker run -p 3003:27017 --name mongo3 --network mongodb-lab2-grupo2 -d mongo:latest
    mongod --replSet rs
```


## 2. Iniciar el Replica Set con los tres nodos

Ahora, debes conectarte al contenedor mongo1 e iniciar el Replica Set, añadiendo los otros dos nodos.

```
docker exec -it mongo1 mongosh
```

<p align="center">
<img src="https://github.com/IAlfaro404/lab2-BDA-MongoDB/blob/main/Requisito%20%231/SalidaTerminalInstalacionManual1.png?raw=true" alt="Este debe ser el resultado, 1." style="display: block; margin: 0 auto;"/></p>

Dentro de la consola mongosh, inicializa el Replica Set:

```
config = {
    "_id": "rs",
    "members": [
    { "_id": 0, "host": "mongo1:27017" },
    { "_id": 1, "host": "mongo2:27017" },
    { "_id": 2, "host": "mongo3:27017" }
    ]
}
```


e inicializar la configuración, luego de esperar máximo 1 minuto.
```
rs.initiate(config)
rs.status ()
```

<details>
  <summary>Esta es la salida que se debe esperar (CLICK!)</summary>

```
rs [direct: secondary] test> rs.status ()
{
  set: 'rs',
  date: ISODate('2025-05-20T21:10:59.725Z'),
  myState: 2,
  term: Long('0'),
  syncSourceHost: '',
  syncSourceId: -1,
  heartbeatIntervalMillis: Long('2000'),
  majorityVoteCount: 2,
  writeMajorityCount: 2,
  votingMembersCount: 3,
  writableVotingMembersCount: 3,
  optimes: {
    lastCommittedOpTime: { ts: Timestamp({ t: 1747775455, i: 1 }), t: Long('-1') },
    lastCommittedWallTime: ISODate('2025-05-20T21:10:55.052Z'),
    readConcernMajorityOpTime: { ts: Timestamp({ t: 1747775455, i: 1 }), t: Long('-1') },
    appliedOpTime: { ts: Timestamp({ t: 1747775455, i: 1 }), t: Long('-1') },
    durableOpTime: { ts: Timestamp({ t: 1747775455, i: 1 }), t: Long('-1') },
    writtenOpTime: { ts: Timestamp({ t: 1747775455, i: 1 }), t: Long('-1') },
    lastAppliedWallTime: ISODate('2025-05-20T21:10:55.052Z'),
    lastDurableWallTime: ISODate('2025-05-20T21:10:55.052Z'),
    lastWrittenWallTime: ISODate('2025-05-20T21:10:55.052Z')
  },
  lastStableRecoveryTimestamp: Timestamp({ t: 1747775455, i: 1 }),
  members: [
    {
      _id: 0,
      name: 'mongo1:27017',
      health: 1,
      state: 2,
      stateStr: 'SECONDARY',
      uptime: 87,
      optime: { ts: Timestamp({ t: 1747775455, i: 1 }), t: Long('-1') },
      optimeDate: ISODate('2025-05-20T21:10:55.000Z'),
      optimeWritten: { ts: Timestamp({ t: 1747775455, i: 1 }), t: Long('-1') },
      optimeWrittenDate: ISODate('2025-05-20T21:10:55.000Z'),
      lastAppliedWallTime: ISODate('2025-05-20T21:10:55.052Z'),
      lastDurableWallTime: ISODate('2025-05-20T21:10:55.052Z'),
      lastWrittenWallTime: ISODate('2025-05-20T21:10:55.052Z'),
      syncSourceHost: '',
      syncSourceId: -1,
      infoMessage: '',
      configVersion: 1,
      configTerm: 0,
      self: true,
      lastHeartbeatMessage: ''
    },
    {
      _id: 1,
      name: 'mongo2:27017',
      health: 1,
      state: 2,
      stateStr: 'SECONDARY',
      uptime: 4,
      optime: { ts: Timestamp({ t: 1747775455, i: 1 }), t: Long('-1') },
      optimeDurable: { ts: Timestamp({ t: 1747775455, i: 1 }), t: Long('-1') },
      optimeWritten: { ts: Timestamp({ t: 1747775455, i: 1 }), t: Long('-1') },
      optimeDate: ISODate('2025-05-20T21:10:55.000Z'),
      optimeDurableDate: ISODate('2025-05-20T21:10:55.000Z'),
      optimeWrittenDate: ISODate('2025-05-20T21:10:55.000Z'),
      lastAppliedWallTime: ISODate('2025-05-20T21:10:55.052Z'),
      lastDurableWallTime: ISODate('2025-05-20T21:10:55.052Z'),
      lastWrittenWallTime: ISODate('2025-05-20T21:10:55.052Z'),
      lastHeartbeat: ISODate('2025-05-20T21:10:59.612Z'),
      lastHeartbeatRecv: ISODate('2025-05-20T21:10:59.308Z'),
      pingMs: Long('0'),
      lastHeartbeatMessage: '',
      syncSourceHost: '',
      syncSourceId: -1,
      infoMessage: '',
      configVersion: 1,
      configTerm: 0
    },
    {
      _id: 2,
      name: 'mongo3:27017',
      health: 1,
      state: 2,
      stateStr: 'SECONDARY',
      uptime: 4,
      optime: { ts: Timestamp({ t: 1747775455, i: 1 }), t: Long('-1') },
      optimeDurable: { ts: Timestamp({ t: 1747775455, i: 1 }), t: Long('-1') },
      optimeWritten: { ts: Timestamp({ t: 1747775455, i: 1 }), t: Long('-1') },
      optimeDate: ISODate('2025-05-20T21:10:55.000Z'),
      optimeDurableDate: ISODate('2025-05-20T21:10:55.000Z'),
      optimeWrittenDate: ISODate('2025-05-20T21:10:55.000Z'),
      lastAppliedWallTime: ISODate('2025-05-20T21:10:55.052Z'),
      lastDurableWallTime: ISODate('2025-05-20T21:10:55.052Z'),
      lastWrittenWallTime: ISODate('2025-05-20T21:10:55.052Z'),
      lastHeartbeat: ISODate('2025-05-20T21:10:59.611Z'),
      lastHeartbeatRecv: ISODate('2025-05-20T21:10:59.333Z'),
      pingMs: Long('0'),
      lastHeartbeatMessage: '',
      syncSourceHost: '',
      syncSourceId: -1,
      infoMessage: '',
      configVersion: 1,
      configTerm: 0
    }
  ],
  ok: 1,
  '$clusterTime': {
    clusterTime: Timestamp({ t: 1747775455, i: 1 }),
    signature: {
      hash: Binary.createFromBase64('AAAAAAAAAAAAAAAAAAAAAAAAAAA=', 0),
      keyId: Long('0')
    }
  },
  operationTime: Timestamp({ t: 1747775455, i: 1 })
}
rs [direct: secondary] test> 
```
</details>


## 3. Crear base de datos `Política` y colección `Discursos` con redundancia activada

Una vez que el Replica Set está configurado y tienes un primario, puedes interactuar con él. Asegúrate de estar conectado al primario (el prompt dirá rs0:PRIMARY>).

```
use Política;
```

En un Replica Set de MongoDB, la redundancia de los datos se activa por defecto para todas las colecciones que se crea, ya que los datos se replican automáticamente entre los miembros. No se necesita un comando especial para "activar la redundancia" a nivel de colección.

```
db.createCollection("Discursos");
```
<p align="center">
<img src="https://github.com/IAlfaro404/lab2-BDA-MongoDB/blob/main/Requisito%20%231/SalidaTerminalInstalacionManual2.png?raw=true" alt="Este debe ser el resultado, 1." style="display: block; margin: 0 auto;"/></p>

## 4. Verificar con rs.status() que los nodos estén correctamente sincronizados

Desde la consola mongosh conectada al nodo primario (mongo1), ejecuta el siguiente comando:

```
rs.status();
```

Qué buscar en la salida:

- `"set": "rs"`: Confirma que estás viendo el estado del Replica Set rs.
- `"members"`: Es un array que lista todos los miembros del Replica Set.
- Para cada miembro (`_id: 0`, `_id: 1`, `_id: 2`), verifica:
    - `"stateStr"`:
        - `PRIMARY:` Indica el nodo primario.
        - `SECONDARY:` Indica los nodos secundarios.
    - `"health": 1`: Significa que el nodo está saludable.
    - `"optimeDate"`: La fecha y hora de la última operación replicada. Deberían ser muy similares entre el primario y los secundarios, indicando que están sincronizados.
    - `"syncFrom"`: Para los secundarios, muestra de qué nodo están replicando.

Si todos los miembros tienen `stateStr` como `PRIMARY` o `SECONDARY` y `health: 1`, y sus optimeDate son cercanos, el clúster MongoDB en Docker está configurado y sincronizado correctamente.



## Consideracion Adicional:
Para asegurar que las escrituras se confirmen por la mayoría de los nodos del Replica Set (garantizando la redundancia y durabilidad), se puede especificar un writeConcern al insertar documentos:

```
db.Discursos.insertOne(
  {
    "_id": "<hash SHA-256>",
    "texto": "<contenido completo>",
    "embedding": [ ... ],
  },
  { writeConcern: { w: "majority" } }
);
