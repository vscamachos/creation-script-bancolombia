# Consultas NoSQL para Sistema Bancario en MongoDB

A continuación se presentan 5 enunciados de consultas basados en las colecciones NoSQL del sistema bancario, junto con las soluciones utilizando operaciones avanzadas de MongoDB.

## 1. Análisis de Saldos por Tipo de Cuenta

**Enunciado:** El departamento financiero necesita un informe que muestre el saldo total, promedio, máximo y mínimo por cada tipo de cuenta (ahorro y corriente) para evaluar la distribución de fondos en el banco.

**Consulta MongoDB:**
```javascript
db.Clientes.aggregate([
  { $unwind: "$cuentas" },
  {
    $group: {
      _id: "$cuentas.tipo_cuenta",
      saldo_total: { $sum: "$cuentas.saldo" },
      saldo_promedio: { $avg: "$cuentas.saldo" },
      saldo_maximo: { $max: "$cuentas.saldo" },
      saldo_minimo: { $min: "$cuentas.saldo" },
      total_cuentas: { $sum: 1 }
    }
  },
  {
    $project: {
      _id: 0,
      tipo_cuenta: "$_id",
      saldo_total: 1,
      saldo_promedio: 1,
      saldo_maximo: 1,
      saldo_minimo: 1,
      total_cuentas: 1
    }
  }
]);
```

## 2. Patrones de Transacciones por Cliente

**Enunciado:** El equipo de análisis de comportamiento necesita identificar los patrones de transacciones de cada cliente, mostrando la cantidad y el monto total de transacciones por tipo (depósito, retiro, transferencia) para cada cliente.

**Consulta MongoDB:**
```javascript
db.Transacciones.aggregate([
  {
    $group: {
      _id: {
        cliente: "$cliente_ref",
        tipo: "$tipo_transaccion"
      },
      cantidad_transacciones: { $sum: 1 },
      monto_total: { $sum: "$monto" }
    }
  },
  {
    $project: {
      _id: 0,
      cliente_id: "$_id.cliente",
      tipo_transaccion: "$_id.tipo",
      cantidad_transacciones: 1,
      monto_total: 1
    }
  }
]);
```

## 3. Clientes con Múltiples Tarjetas de Crédito

**Enunciado:** El departamento de riesgo crediticio necesita identificar a los clientes que poseen más de una tarjeta de crédito, mostrando sus datos personales, cantidad de tarjetas y el detalle de cada una.

**Consulta MongoDB:**
```javascript
db.Clientes.aggregate([
  { $unwind: "$cuentas" },
  { $unwind: "$cuentas.tarjetas" },
  {
    $project: {
      _id: 0,
      nombre: 1,
      cedula: 1,
      correo: 1,
      tipo_tarjeta: "$cuentas.tarjetas.tipo_tarjeta",
      numero_tarjeta: "$cuentas.tarjetas.numero_tarjeta"
    }
  }
]);
```

## 4. Análisis de Medios de Pago más Utilizados

**Enunciado:** El departamento de marketing necesita conocer cuáles son los medios de pago más utilizados para depósitos, agrupados por mes, para orientar sus campañas promocionales.

**Consulta MongoDB:**
```javascript
db.Transacciones.aggregate([
  { $match: { tipo_transaccion: "deposito" } },
  {
    $project: {
      medio_pago: "$detalles_deposito.medio_pago",  // Extraemos el medio de pago
      fecha: { $toDate: "$fecha" }  // Convertimos el campo fecha a tipo Date si es necesario
    }
  },
  {
    $project: {
      medio_pago: "$detalles_deposito.medio_pago",
      mes: { $dateToString: { format: "%Y-%m", date: "$fecha" } }  // Extraemos solo el mes y año (YYYY-MM)
    }
  },
  {
    $group: {
      _id: { mes: "$mes", medio_pago: "$medio_pago" },
      cantidad: { $sum: 1 }
    }
  },
  { $sort: { "_id.mes": 1, "cantidad": -1 } },
  {
    $project: {
      _id: 0,
      mes: "$_id.mes",
      medio_pago: "$_id.medio_pago",
      cantidad: 1
    }
  }
]);
```

## 5. Detección de Cuentas con Transacciones Sospechosas

**Enunciado:** El departamento de seguridad necesita identificar cuentas con patrones de transacciones sospechosas, definidas como aquellas que tienen más de 3 retiros en un mismo día con un monto total superior a 1,000,000 COP.

**Consulta MongoDB:**
```javascript
db.Transacciones.aggregate([
  {
    $match: { tipo_transaccion: "retiro" }
  },
  {
    $project: {
      num_cuenta: 1,
      monto: 1,
      fecha: { $toDate: "$fecha" },  // Convertir fecha a Date si es necesario
      fecha_sin_hora: { 
        $dateToString: { format: "%Y-%m-%d", date: { $toDate: "$fecha" } }  // Extraer solo la fecha (YYYY-MM-DD)
      }
    }
  },
  {
    $group: {
      _id: { num_cuenta: "$num_cuenta", fecha: "$fecha_sin_hora" },
      cantidad_retiros: { $sum: 1 },
      monto_total_retiros: { $sum: "$monto" }
    }
  },
  {
    $match: {
      cantidad_retiros: { $gt: 3 },
      monto_total_retiros: { $gt: 1000000 }
    }
  },
  {
    $project: {
      num_cuenta: "$_id.num_cuenta",
      fecha: "$_id.fecha",
      cantidad_retiros: 1,
      monto_total_retiros: 1,
      _id: 0
    }
  },
  { $sort: { "num_cuenta": 1, "fecha": 1 } }
]);
```
