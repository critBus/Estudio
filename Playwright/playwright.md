# general

Es para hacer test desde una perspectiva externa al objetivo (simular que somos un usuario real)

Tiene implementaciones en varios idiomas, pero nos vamos a centrar en typescript



# instalacion

Dentro de la carpeta del proyecto a probar /MiProyecto

```bash
pnpm create playwright
```



el va a instalarce con el node modules y va a crear una carpeta /tests y algunos archivos



Los archivos de test se van a inlcuir la terminacion `.spec.ts` ejemplo mi-test.spec.ts



# Ejecutar los test



```bash
pnpm exec playwright test
```



al final te redirige a una pestaña donde se ven los resultados `http://localhost:9323/`


