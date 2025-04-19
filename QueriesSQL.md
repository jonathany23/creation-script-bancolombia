# Consultas SQL para Base de Datos Bancaria

## Enunciado 1: Clientes con múltiples cuentas y sus saldos totales

**Necesidad:** El banco necesita identificar a los clientes que tienen más de una cuenta, mostrar cuántas cuentas tiene cada uno y el saldo total acumulado en todas sus cuentas, ordenados por saldo total de mayor a menor.

**Consulta SQL:**
```sql
SELECT
    c.nombre, c.cedula, COUNT(cu.num_cuenta) AS cantidad_cuentas, SUM(cu.saldo) AS saldo_total_acumulado
FROM
    sqlcourse.cliente c
JOIN
    sqlcourse.cuenta cu ON c.id_cliente = cu.id_cliente
GROUP BY
    c.id_cliente, c.nombre, c.cedula
HAVING
    COUNT(cu.num_cuenta) > 1
ORDER BY
    saldo_total_acumulado DESC;
```

## Enunciado 2: Comparativa entre depósitos y retiros por cliente

**Necesidad:** El departamento de análisis financiero necesita comparar los montos totales de depósitos y retiros realizados por cada cliente, para identificar patrones de comportamiento financiero.

**Consulta SQL:**
```sql
SELECT
    c.nombre,
    c.cedula,
    SUM(CASE WHEN t.tipo_transaccion = 'deposito' THEN t.monto ELSE 0 END) AS total_depositos,
    SUM(CASE WHEN t.tipo_transaccion = 'retiro' THEN t.monto ELSE 0 END) AS total_retiros
FROM
    cliente c
JOIN
    cuenta cu ON c.id_cliente = cu.id_cliente
JOIN
    transaccion t ON cu.num_cuenta = t.num_cuenta
GROUP BY
    c.id_cliente, c.nombre, c.cedula
ORDER BY
    c.id_cliente;
```

## Enunciado 3: Cuentas sin tarjetas asociadas

**Necesidad:** El departamento de tarjetas necesita identificar todas las cuentas que no tienen tarjetas asociadas para ofrecer productos específicos a estos clientes.

**Consulta SQL:**
```sql
SELECT
    cu.num_cuenta,
    cu.id_cliente,
    cu.tipo_cuenta,
    cu.saldo,
    cu.fecha_apertura
FROM
    sqlcourse.cuenta cu
LEFT JOIN
    sqlcourse.tarjeta ta ON cu.num_cuenta = ta.num_cuenta
WHERE
    ta.num_cuenta IS NULL;
```

## Enunciado 4: Análisis de saldos promedio por tipo de cuenta y comportamiento transaccional

**Necesidad:** La gerencia necesita un análisis comparativo del saldo promedio entre cuentas de ahorro y corriente, pero solo considerando aquellas cuentas que han tenido al menos una transacción en los últimos 30 días.

**Consulta SQL:**
```sql
SELECT
    cu.tipo_cuenta,
    AVG(cu.saldo) AS saldo_promedio
FROM
    cuenta cu
WHERE
    EXISTS (
        SELECT 1
        FROM
            transaccion t
        WHERE
            t.num_cuenta = cu.num_cuenta
            AND t.fecha >= CURRENT_DATE - INTERVAL '30 days'
    )
GROUP BY
    cu.tipo_cuenta;
```

## Enunciado 5: Clientes con transferencias pero sin retiros en cajeros

**Necesidad:** El equipo de marketing digital necesita identificar a los clientes que utilizan transferencias pero no realizan retiros por cajeros automáticos, para dirigir campañas de banca digital.

**Consulta SQL:**
```sql
SELECT
    c.id_cliente,
    c.nombre,
    c.cedula,
    c.correo
FROM
    cliente c
JOIN
    cuenta cu ON c.id_cliente = cu.id_cliente
WHERE
    EXISTS (
        SELECT 1
        FROM
            transaccion t
        WHERE
            t.num_cuenta = cu.num_cuenta
            AND t.tipo_transaccion = 'transferencia'
    )
    AND NOT EXISTS (
        SELECT 1
        FROM
            transaccion t2
        WHERE
            t2.num_cuenta = cu.num_cuenta
            AND t2.tipo_transaccion = 'retiro'
            and t2.descripcion = 'Retiro en cajero automático'
    )
GROUP BY
    c.id_cliente, c.nombre, c.cedula, c.correo;
```
