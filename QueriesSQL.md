# Consultas SQL para Base de Datos Bancaria

## Enunciado 1: Clientes con múltiples cuentas y sus saldos totales

**Necesidad:** El banco necesita identificar a los clientes que tienen más de una cuenta, mostrar cuántas cuentas tiene cada uno y el saldo total acumulado en todas sus cuentas, ordenados por saldo total de mayor a menor.

**Consulta SQL:**
```sql
select 
    cl.id_cliente,
    cl.nombre,
    count(cta.num_cuenta) as numcuentas,
    sum(cta.saldo) as saldototal
from cliente cl
join cuenta cta 
	on cl.id_cliente = cta.id_cliente
group by cl.id_cliente, cl.nombre
having count(cta.num_cuenta) > 1
order by saldototal desc;
```

## Enunciado 2: Comparativa entre depósitos y retiros por cliente

**Necesidad:** El departamento de análisis financiero necesita comparar los montos totales de depósitos y retiros realizados por cada cliente, para identificar patrones de comportamiento financiero.

**Consulta SQL:**
```sql
select 
    cl.id_cliente,
    cl.nombre,
    sum(case when t.tipo_transaccion = 'deposito' then t.monto else 0 end) as totaldepositos,
    sum(case when t.tipo_transaccion = 'retiro' then t.monto else 0 end) as totalretiros
from cliente cl
left join cuenta cta 
	on cl.id_cliente = cta.id_cliente
left join transaccion t 
	on cta.num_cuenta = t.num_cuenta
group by cl.id_cliente, cl.nombre
order by totaldepositos desc;
```

## Enunciado 3: Cuentas sin tarjetas asociadas

**Necesidad:** El departamento de tarjetas necesita identificar todas las cuentas que no tienen tarjetas asociadas para ofrecer productos específicos a estos clientes.

**Consulta SQL:**
```sql
select 
    cta.num_cuenta,
    cta.id_cliente
from cuenta cta
left join tarjeta t 
	on cta.num_cuenta = t.num_cuenta
where t.numero_tarjeta is null;
```

## Enunciado 4: Análisis de saldos promedio por tipo de cuenta y comportamiento transaccional

**Necesidad:** La gerencia necesita un análisis comparativo del saldo promedio entre cuentas de ahorro y corriente, pero solo considerando aquellas cuentas que han tenido al menos una transacción en los últimos 30 días.

**Consulta SQL:**
```sql
select 
    cta.tipo_cuenta,
    avg(cta.saldo) as saldo_promedio
from cuenta cta
join transaccion t 
	on cta.num_cuenta = t.num_cuenta
where t.fecha >= '2023-03-31'::date - interval '30 day' -- la ultima fecha encontrada en la tabla se le restan 30 días
group by cta.tipo_cuenta;
```

## Enunciado 5: Clientes con transferencias pero sin retiros en cajeros

**Necesidad:** El equipo de marketing digital necesita identificar a los clientes que utilizan transferencias pero no realizan retiros por cajeros automáticos, para dirigir campañas de banca digital.

**Consulta SQL:**
```sql
select distinct cl.id_cliente, cl.nombre
from cliente cl
join cuenta cta 
	on cl.id_cliente = cta.id_cliente
join transaccion t 
	on cta.num_cuenta = t.num_cuenta
where t.tipo_transaccion = 'transferencia'
and cl.id_cliente not in (
    select distinct cl2.id_cliente
    from cliente cl2
    join cuenta cta2 
		on cl2.id_cliente = cta2.id_cliente
    join transaccion t2 
		on cta2.num_cuenta = t2.num_cuenta
    where t2.descripcion = 'retiro en cajero'
);
```
