# Parte 6: Buenas prácticas del diseño de redes

## 11. ¿Qué es lo recomendable a la hora de diseñar una infraestructura DNS? ¿A qué se refiere con el principio de separación estricta de funciones?

Lo recomendable es no usar un solo servidor DNS para todo, sino armar la infraestructura con roles separados, tener redundancia (minimo dos servidores por zona, y si se puede en ubicaciones distintas) y exponer lo menos posible a Internet

El principio de separación estricta de funciones quiere decir que cada servidor DNS tiene que cumplir una sola función:

- Autoritativos: solo responden por las zonas que administra la organización (por ejemplo empresa.com)
- Recursivos (resolvers): resuelven las consultas de los clientes internos hacia cualquier dominio de Internet.

Se hace asi porque si se mezclan las dos funciones en el mismo servidor aumenta la superficie de ataque. Un recursivo expuesto a Internet se puede usar para ataques DDoS de amplificación/reflexión y para envenenamiento de cache (cache poisoning). Separandolos, si le pasa algo a uno no afecta al otro y además se pueden aplicar politicas de seguridad distintas.

## 12. ¿Qué atienden los servidores autoritativos públicos? ¿Qué deberían tener totalmente desactivado?

Atienden unicamente las consultas que llegan desde Internet sobre las zonas públicas de la organización (web, correo/MX, etc).

Lo que deberían tener totalmente desactivado es la recursión. Un autoritativo público solo responde por sus propias zonas, si le preguntan por un dominio ajeno tiene que rechazar la consulta. En BIND sería asi:

```
options {
    recursion no;
};
```

Tambien conviene limitar las transferencias de zona (`allow-transfer`) solo a los secundarios autorizados.

## 13. ¿Qué función cumplen los servidores recursivos internos o resolvers? ¿Con qué opción se restringe únicamente a los rangos IP de la organización?

Atienden las consultas de los clientes internos (PCs, servidores, dispositivos). Resuelven nombres de cualquier dominio preguntandole a otros servidores DNS en nombre del cliente, y guardan el resultado en cache para ir mas rapido en futuras consultas y gastar menos trafico externo. Tambien pueden resolver zonas internas privadas.

La opción para restringirlo a los rangos IP de la organización es `allow-recursion`, que se usa con una ACL con las redes internas. Ejemplo en BIND:

```
acl "redes_internas" {
    192.168.0.0/16;
    10.0.0.0/8;
    127.0.0.1;
};

options {
    recursion yes;
    allow-recursion { redes_internas; };
};
```

Asi el resolver solo responde a clientes de la organización y no queda como open resolver para todo Internet. Se suele complementar con `allow-query-cache` para limitar tambien quien puede consultar la cache
