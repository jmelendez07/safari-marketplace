# 09. Cambiar las contraseñas demo (obligatorio)

[← 08. Desplegar y comprobar](08-desplegar-y-comprobar.md) · [Índice](README.md) · [10. Publicar una versión nueva o volver atrás →](10-actualizar-y-volver-atras.md)

---

Los datos iniciales traen cinco usuarios con la **misma contraseña pública**,
`demodemo`, que figura en este repositorio. Con el admin en internet,
cualquiera podría entrar. **Cámbialas apenas termine el primer despliegue.**

| Usuario | Rol |
|---|---|
| `admin@demo.com` | super_admin |
| `store_owner@demo.com` | store_owner |
| `customer@demo.com` | customer |
| `customer2@demo.com` | customer |
| `customer3@demo.com` | customer |

Desde tu equipo, cambia `<tu-dominio>`. El script pregunta la contraseña **actual**
(Enter si todavía es `demodemo`) y la nueva:

```bash
TIENDA=https://tienda.<tu-dominio>
for USUARIO in admin@demo.com store_owner@demo.com customer@demo.com customer2@demo.com customer3@demo.com; do
  read -rsp "Contraseña actual de $USUARIO [demodemo]: " ACTUAL; echo
  ACTUAL=${ACTUAL:-demodemo}
  read -rsp "Contraseña nueva para $USUARIO: " NUEVA; echo

  TOKEN=$(curl -s -X POST "$TIENDA/backend/token" -H 'Content-Type: application/json' \
    -d "{\"email\":\"$USUARIO\",\"password\":\"$ACTUAL\"}" \
    | sed -n 's/.*"token":"\([^"]*\)".*/\1/p')

  if [ -z "$TOKEN" ]; then
    echo "$USUARIO: contraseña actual incorrecta, no se cambió nada"
    continue
  fi

  RESPUESTA=$(curl -s -X POST "$TIENDA/backend/change-password" \
    -H 'Content-Type: application/json' -H "Authorization: Bearer $TOKEN" \
    -d "{\"oldPassword\":\"$ACTUAL\",\"newPassword\":\"$NUEVA\"}")

  case "$RESPUESTA" in
    *'"success":true'*) echo "$USUARIO: contraseña cambiada" ;;
    *) echo "$USUARIO: NO se cambió — $RESPUESTA" ;;
  esac
done
```

Cada usuario debe terminar en «contraseña cambiada».

> **Si ya cambiaste una contraseña**, escribe la nueva en «Contraseña actual».
> El `401` significa que el login falló porque `demodemo` ya no es válida; no es
> que la contraseña nueva esté mal. `demodemo` solo funciona la primera vez.

Comprueba que `demodemo` ya no sirve:

```bash
curl -s -o /dev/null -w "%{http_code}\n" -X POST "$TIENDA/backend/token" \
  -H 'Content-Type: application/json' -d '{"email":"admin@demo.com","password":"demodemo"}'
```

Las contraseñas nuevas se guardan en la base y los despliegues siguientes no
las tocan, porque el seed no se vuelve a aplicar.

---

[← 08. Desplegar y comprobar](08-desplegar-y-comprobar.md) · [Índice](README.md) · [10. Publicar una versión nueva o volver atrás →](10-actualizar-y-volver-atras.md)
