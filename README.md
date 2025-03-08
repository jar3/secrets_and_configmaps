# ¿Qué son los ConfigMaps y los Secrets en Kubernetes?

Imagina que tienes una cafetería y necesitas administrar información de dos tipos:

## Información pública:
- El menú del día, el horario de apertura, los ingredientes de los productos.
- No es secreto, cualquiera puede verlo.
- Lo anotas en una pizarra o lo guardas en una hoja.
- **En Kubernetes, esto es un ConfigMap.**

## Información privada:
- La clave del Wi-Fi para empleados, la contraseña de la caja registradora, la receta secreta de un café especial.
- No debe ser accesible para todos.
- Lo guardas en una caja fuerte o en un papel bien escondido.
- **En Kubernetes, esto es un Secret.**

Ambos se usan para configurar el funcionamiento de la cafetería (o de una aplicación en Kubernetes), pero la diferencia es que los Secrets están protegidos porque contienen datos sensibles.

## 📌 Ejemplo con una Aplicación en Kubernetes

Supongamos que tienes una aplicación que necesita conectarse a una base de datos:
- El ConfigMap guardará información no sensible, como la URL de la base de datos.
- El Secret guardará credenciales como el usuario y la contraseña.

### 📝 ConfigMap (Información Pública)

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: cafe-config
data:
  menu: "Café, Té, Chocolate caliente"
  horario: "08:00 - 20:00"
```

Este ConfigMap puede ser usado por los pods para leer el menú y el horario.

### 🔑 Secret (Información Privada)

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: cafe-secret
type: Opaque
data:
  wifi_password: Y2FmZVdpRmkwMDE=  # Base64 de "cafeWiFi001"
  caja_registradora_pin: MTIzNA==  # Base64 de "1234"
```

Aquí, las contraseñas están codificadas en base64 para que no sean legibles a simple vista.

## 🎯 Práctica en Minikube

Vamos a probar esto en tu clúster Minikube.

### 1️⃣ Crea el ConfigMap y el Secret

Ejecuta estos comandos en tu terminal:

```bash
kubectl create configmap cafe-config --from-literal=menu="Café, Té, Chocolate caliente" --from-literal=horario="08:00 - 20:00"
kubectl create secret generic cafe-secret --from-literal=wifi_password="cafeWiFi001" --from-literal=caja_registradora_pin="1234"
```

### 2️⃣ Verifica que se crearon correctamente

```bash
kubectl get configmap cafe-config -o yaml
kubectl get secret cafe-secret -o yaml
```

Verás que en el Secret los valores están codificados en Base64.

### 3️⃣ Crea un Pod que use el ConfigMap y el Secret

Guarda el siguiente YAML en un archivo llamado cafe-pod.yaml:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: cafe-pod
spec:
  containers:
    - name: cafe-container
      image: busybox
      command: ["sleep", "3600"]
      env:
        - name: MENU_DEL_DIA
          valueFrom:
            configMapKeyRef:
              name: cafe-config
              key: menu
        - name: WIFI_PASSWORD
          valueFrom:
            secretKeyRef:
              name: cafe-secret
              key: wifi_password
```

Aplica el pod con:

```bash
kubectl apply -f cafe-pod.yaml
```

### 4️⃣ Comprueba que el pod tiene las variables

```bash
kubectl exec -it cafe-pod -- env | grep MENU_DEL_DIA
kubectl exec -it cafe-pod -- env | grep WIFI_PASSWORD
```

Aquí verás que MENU_DEL_DIA se muestra tal cual y WIFI_PASSWORD también, aunque en una aplicación real se manejaría con más seguridad.

## 🚀 Conclusión

- ConfigMaps son como pizarras con información pública.
- Secrets son cajas fuertes con información sensible.
- Se pueden usar en pods como variables de entorno o montados como archivos.
