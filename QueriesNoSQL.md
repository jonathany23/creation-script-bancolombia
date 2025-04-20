# Consultas NoSQL para Sistema Bancario en MongoDB

A continuación se presentan 5 enunciados de consultas basados en las colecciones NoSQL del sistema bancario, junto con las soluciones utilizando operaciones avanzadas de MongoDB.

## 1. Análisis de Saldos por Tipo de Cuenta

**Enunciado:** El departamento financiero necesita un informe que muestre el saldo total, promedio, máximo y mínimo por cada tipo de cuenta (ahorro y corriente) para evaluar la distribución de fondos en el banco.

**Consulta MongoDB:**
```javascript
db.getCollection('Clientes').aggregate(
  [
    { $unwind: '$cuentas' },
    {
      $group: {
        _id: '$cuentas.tipo_cuenta',
        totalSaldo: { $sum: '$cuentas.saldo' },
        promedioSaldo: { $avg: '$cuentas.saldo' },
        maximoSaldo: { $max: '$cuentas.saldo' },
        minimoSaldo: { $min: '$cuentas.saldo' },
        count: { $sum: 1 }
      }
    },
    {
      $project: {
        _id: 0,
        tipo_cuenta: '$_id',
        totalSaldo: 1,
        promedioSaldo: {
          $round: ['$promedioSaldo', 2]
        },
        maximoSaldo: 1,
        minimoSaldo: 1,
        count: 1
      }
    }
  ]
);
```

## 2. Patrones de Transacciones por Cliente

**Enunciado:** El equipo de análisis de comportamiento necesita identificar los patrones de transacciones de cada cliente, mostrando la cantidad y el monto total de transacciones por tipo (depósito, retiro, transferencia) para cada cliente.

**Consulta MongoDB:**
```javascript
db.getCollection('Transacciones').aggregate(
  [
    {
      $group: {
        _id: {
          clienteId: '$cliente_ref',
          tipoTransaccion: '$tipo_transaccion'
        },
        cantidad: { $sum: 1 },
        montoTotal: { $sum: '$monto' }
      }
    },
    {
      $group: {
        _id: '$_id.clienteId',
        resumenPorTipo: {
          $push: {
            tipo: '$_id.tipoTransaccion',
            cantidad: '$cantidad',
            montoTotal: '$montoTotal'
          }
        }
      }
    },
    {
      $lookup: {
        from: 'Clientes',
        localField: '_id',
        foreignField: '_id',
        as: 'cliente'
      }
    },
    { $unwind: '$cliente' },
    {
      $project: {
        _id: 0,
        clienteId: '$_id',
        nombre: '$cliente.nombre',
        cedula: '$cliente.cedula',
        transacciones: '$resumenPorTipo'
      }
    }
  ]
);
```

## 3. Clientes con Múltiples Tarjetas de Crédito

**Enunciado:** El departamento de riesgo crediticio necesita identificar a los clientes que poseen más de una tarjeta de crédito, mostrando sus datos personales, cantidad de tarjetas y el detalle de cada una.

**Consulta MongoDB:**
```javascript
db.getCollection('Clientes').aggregate(
  [
    { $unwind: '$cuentas' },
    { $unwind: '$cuentas.tarjetas' },
    {
      $match: {
        'cuentas.tarjetas.tipo_tarjeta': 'credito'
      }
    },
    {
      $group: {
        _id: '$_id',
        nombre: { $first: '$nombre' },
        cedula: { $first: '$cedula' },
        correo: { $first: '$correo' },
        direccion: { $first: '$direccion' },
        tarjetas_credito: {
          $push: '$cuentas.tarjetas'
        },
        cantidad_tarjetas_credito: { $sum: 1 }
      }
    },
    {
      $match: {
        cantidad_tarjetas_credito: { $gt: 1 }
      }
    },
    {
      $project: {
        _id: 0,
        nombre: 1,
        cedula: 1,
        correo: 1,
        direccion: 1,
        cantidad_tarjetas_credito: 1,
        tarjetas_credito: 1
      }
    }
  ]
);
```

## 4. Análisis de Medios de Pago más Utilizados

**Enunciado:** El departamento de marketing necesita conocer cuáles son los medios de pago más utilizados para depósitos, agrupados por mes, para orientar sus campañas promocionales.

**Consulta MongoDB:**
```javascript
db.getCollection('Transacciones').aggregate(
  [
    { $match: { tipo_transaccion: 'deposito' } },
    {
      $addFields: {
        mes: { $month: { $toDate: '$fecha' } }
      }
    },
    {
      $group: {
        _id: {
          mes: '$mes',
          medio_pago:
            '$detalles_deposito.medio_pago'
        },
        cantidad: { $sum: 1 }
      }
    },
    { $sort: { '_id.mes': 1, cantidad: -1 } },
    {
      $group: {
        _id: '$_id.mes',
        medios_de_pago: {
          $push: {
            medio_pago: '$_id.medio_pago',
            cantidad: '$cantidad'
          }
        }
      }
    },
    {
      $project: {
        _id: 0,
        mes: '$_id',
        medios_de_pago: 1
      }
    }
  ]
);
```

## 5. Detección de Cuentas con Transacciones Sospechosas

**Enunciado:** El departamento de seguridad necesita identificar cuentas con patrones de transacciones sospechosas, definidas como aquellas que tienen más de 3 retiros en un mismo día con un monto total superior a 1,000,000 COP.

**Consulta MongoDB:**
```javascript
db.getCollection('Transacciones').aggregate(
  [
    {
      $match: {
        tipo_transaccion: 'retiro',
        monto: { $gt: 0 }
      }
    },
    {
      $group: {
        _id: {
          num_cuenta: '$num_cuenta',
          fecha_dia: {
            $dateToString: {
              format: '%Y-%m-%d',
              date: { $toDate: '$fecha' }
            }
          }
        },
        total_retiros: { $sum: 1 },
        monto_total_dia: { $sum: '$monto' }
      }
    },
    {
      $match: {
        total_retiros: { $gt: 3 },
        monto_total_dia: { $gt: 1000000 }
      }
    },
    {
      $project: {
        _id: 0,
        num_cuenta: '$_id.num_cuenta',
        fecha_dia: '$_id.fecha_dia',
        total_retiros: 1,
        monto_total_dia: 1
      }
    }
  ]
);
```
