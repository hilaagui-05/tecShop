# Week #11 - Activity: Detectalo y luego dirigí a la IA

## Paso 1: Mapa de problemas

| #  | Dónde (clase.método)                          | Síntoma: ¿qué va a doler cuando el código cambie?                                                                                                              | ¿Quién sabe demasiado?          | ¿Qué clase(s) existente(s) dejarían intactas?                                        |
| -- | --------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------- | ------------------------------------------------------------------------------------ |
| S1 | ShippingService.quote()                     | El servicio conoce la API de Correos, realiza conversiones y procesa su respuesta. Si cambia la API o se agrega otro proveedor, hay que modificar el servicio. | ShippingService               | CorreosApi y LocalCourier                                                        |
| S2 | CartItem.getPrice() y CartItem.describe() | Los extras se manejan con atributos booleanos y condiciones. Cada extra nuevo obliga a modificar la clase y su lógica.                                         | CartItem                      | Las clases de los servicios y el proceso de compra.                                  |
| S3 | WebCheckout.buy() y KioskCheckout.buy()   | Los dos métodos repiten la reserva, el cálculo de impuestos, el envío, el pago y la impresión de la factura.                                                   | WebCheckout y KioskCheckout | Inventory, TaxCalculator, ShippingService, PaymentGateway e InvoicePrinter |


## Ronda 1: Envíos

### Prompt 1: Crear CorreosShipping

#### Prompt enviado

CONTEXTO: En el paquete tecshop existen ShippingProvider.java y CorreosApi.java. ShippingProvider declara el método double quote(double kg, String city). CorreosApi es una biblioteca externa que no se puede modificar.

TAREA: Crea un archivo nuevo llamado CorreosShipping.java.

ESTRUCTURA: La clase CorreosShipping implementa ShippingProvider. Tiene un único atributo private final CorreosApi api, inicializado mediante un constructor que recibe un objeto CorreosApi. Por ahora, el método quote(double kg, String city) debe lanzar UnsupportedOperationException.

REGLAS: Mantén el paquete tecshop. No modifiques CorreosApi.java, ShippingProvider.java ni ningún otro archivo. No agregues bibliotecas.

TERMINADO: El proyecto compila con el comando javac -encoding UTF-8 -d out *.java. Mostrame únicamente el contenido completo de CorreosShipping.j
#### Evaluación de la respuesta

La IA creó la clase solicitada, implementó ShippingProvider y declaró el atributo api como privado y final. También agregó el constructor y dejó temporalmente el método quote con UnsupportedOperationException, tal como se indicó. No modificó otras clases.

#### Salida obtenida

```java
package tecshop;

public class CorreosShipping implements ShippingProvider {
    private final CorreosApi api;

    public CorreosShipping(CorreosApi api) {
        this.api = api;
    }

    @Override
    public double quote(double kg, String city) {
        throw new UnsupportedOperationException();
    }
}
```

#### Resultado de compilación

```text
PS C:\TEC\2-Semestre\POO\tecshop> javac -encoding UTF-8 -d out *.java
PS C:\TEC\2-Semestre\POO\tecshop> java -cp out tecshop.Main
Inventory: reserved 3 items
Payment: charged CRC 61,832.00 via credit card
----- INVOICE: Ana -----
  Laptop stand + gift wrap + express  CRC 29,000.00
  USB-C hub + insurance  CRC 18,900.00
  Notebook  CRC 3,500.00
  Subtotal: CRC 51,400.00
  Tax:      CRC 6,682.00
  Shipping: CRC 3,750.00
  TOTAL:    CRC 61,832.00

Inventory: reserved 1 items
Payment: charged CRC 53,592.75 via SINPE Movil
----- INVOICE: Luis -----
  Mechanical keyboard + gift wrap + insurance  CRC 45,675.00
  Subtotal: CRC 45,675.00
  Tax:      CRC 5,937.75
  Shipping: CRC 1,980.00
  TOTAL:    CRC 53,592.75
```

### Prompt 2: Completar CorreosShipping

#### Prompt enviado

CONTEXTO: En el paquete tecshop existe CorreosShipping.java, que implementa ShippingProvider y tiene el atributo private final CorreosApi api, recibido por constructor. La interfaz declara double quote(double kg, String city). La clase externa CorreosApi ofrece el método String calcularTarifa(int gramos, String codigoPostal).

TAREA: Completa únicamente el método quote de CorreosShipping.java.

ESTRUCTURA: Convierte los kilogramos a gramos mediante (int) Math.round(kg * 1000). Obtén el código postal según la ciudad: San Jose = "10101", Alajuela = "20101", Cartago = "30101" y Limon = "70101". Si la ciudad no está contemplada, lanza IllegalArgumentException con el mensaje "Unknown city: " + city. Llama a api.calcularTarifa(gramos, codigoPostal), recibe el texto devuelto y elimina el símbolo "₡" para convertir el resultado a double mediante Double.parseDouble.

REGLAS: No modifiques CorreosApi.java, ShippingProvider.java, ShippingService.java ni ningún otro archivo. No agregues bibliotecas. Mantén el paquete tecshop y el atributo existente.

TERMINADO: El proyecto compila con javac -encoding UTF-8 -d out *.java. Mostrame únicamente el contenido completo de CorreosShipping.java.

#### Evaluación de la respuesta

La IA completó el método `quote` con la conversión de kilogramos a gramos, la selección del código postal y el tratamiento de ciudades no contempladas. También utiliza la API externa y convierte su respuesta textual a un número. La clase `CorreosApi` permanece sin modificaciones.

#### Salida obtenida

```java
package tecshop;

public class CorreosShipping implements ShippingProvider {
    private final CorreosApi api;

    public CorreosShipping(CorreosApi api) {
        this.api = api;
    }

    @Override
    public double quote(double kg, String city) {
        int gramos = (int) Math.round(kg * 1000);
        String codigoPostal;

        switch (city) {
            case "San Jose": codigoPostal = "10101"; break;
            case "Alajuela": codigoPostal = "20101"; break;
            case "Cartago": codigoPostal = "30101"; break;
            case "Limon": codigoPostal = "70101"; break;
            default: throw new IllegalArgumentException("Unknown city: " + city);
        }

        String tarifa = api.calcularTarifa(gramos, codigoPostal);
        return Double.parseDouble(tarifa.replace("₡", ""));
    }
}
```

#### Resultado de compilación

```text
PS C:\TEC\2-Semestre\POO\tecshop> javac -encoding UTF-8 -d out *.java
PS C:\TEC\2-Semestre\POO\tecshop> java -cp out tecshop.Main
Inventory: reserved 3 items
Payment: charged CRC 61,832.00 via credit card
----- INVOICE: Ana -----
  Laptop stand + gift wrap + express  CRC 29,000.00
  USB-C hub + insurance  CRC 18,900.00
  Notebook  CRC 3,500.00
  Subtotal: CRC 51,400.00
  Tax:      CRC 6,682.00
  Shipping: CRC 3,750.00
  TOTAL:    CRC 61,832.00

Inventory: reserved 1 items
Payment: charged CRC 53,592.75 via SINPE Movil
----- INVOICE: Luis -----
  Mechanical keyboard + gift wrap + insurance  CRC 45,675.00
  Subtotal: CRC 45,675.00
  Tax:      CRC 5,937.75
  Shipping: CRC 1,980.00
  TOTAL:    CRC 53,592.75
```

---

## Ronda 2: Extras

### Prompt 1: Crear CartProduct

#### Prompt enviado

CONTEXTO: En el paquete tecshop existe la clase CartItem, que representa un artículo y tiene los métodos getPrice() y describe(). Los métodos calculan su precio y generan su descripción. La clase debe seguir funcionando como hasta ahora.

TAREA: Crea un archivo nuevo llamado CartProduct.java.

ESTRUCTURA: Declara en el paquete tecshop una interfaz pública CartProduct con los métodos double getPrice() y String describe().

REGLAS: No modifiques ningún archivo existente. No agregues bibliotecas.

TERMINADO: El proyecto compila con javac -encoding UTF-8 -d out *.java. Mostrame únicamente el contenido completo de CartProduct.java.

#### Evaluación de la respuesta

Completar después de comprobar que la interfaz contiene ambos métodos y que no se modificaron otros archivos.

#### Salida obtenida

[Pegar `CartProduct.java`.]

#### Resultado de compilación

```text
PS C:\TEC\2-Semestre\POO\tecshop> javac -encoding UTF-8 -d out *.java
PS C:\TEC\2-Semestre\POO\tecshop> java -cp out tecshop.Main
Inventory: reserved 3 items
Payment: charged CRC 61,832.00 via credit card
----- INVOICE: Ana -----
  Laptop stand + gift wrap + express  CRC 29,000.00
  USB-C hub + insurance  CRC 18,900.00
  Notebook  CRC 3,500.00
  Subtotal: CRC 51,400.00
  Tax:      CRC 6,682.00
  Shipping: CRC 3,750.00
  TOTAL:    CRC 61,832.00

Inventory: reserved 1 items
Payment: charged CRC 53,592.75 via SINPE Movil
----- INVOICE: Luis -----
  Mechanical keyboard + gift wrap + insurance  CRC 45,675.00
  Subtotal: CRC 45,675.00
  Tax:      CRC 5,937.75
  Shipping: CRC 1,980.00
  TOTAL:    CRC 53,592.75
```

### Prompt 2: Hacer que CartItem implemente CartProduct

#### Prompt enviado

CONTEXTO: En el paquete tecshop existe la interfaz CartProduct, con los métodos double getPrice() y String describe(). La clase CartItem ya implementa esos comportamientos mediante sus métodos actuales.

TAREA: Modifica únicamente CartItem.java para que implemente CartProduct.

ESTRUCTURA: Conserva el comportamiento actual de getPrice() y describe(), todos los cálculos, el orden de los extras, los constructores existentes y los atributos existentes. Agrega las anotaciones @Override a los métodos que implementan la interfaz.

REGLAS: No modifiques ningún otro archivo. No cambies los precios ni la salida del programa. No agregues bibliotecas.

TERMINADO: Compila y ejecuta Main. La salida debe coincidir exactamente con la salida base. Mostrame únicamente CartItem.java completo.

#### Evaluación de la respuesta

Completar después de verificar que `CartItem` implementa la interfaz y que conserva el comportamiento original.

#### Salida obtenida

[Pegar `CartItem.java`.]

#### Resultado de compilación

```text
PS C:\TEC\2-Semestre\POO\tecshop> javac -encoding UTF-8 -d out *.java
PS C:\TEC\2-Semestre\POO\tecshop> java -cp out tecshop.Main
Inventory: reserved 3 items
Payment: charged CRC 61,832.00 via credit card
----- INVOICE: Ana -----
  Laptop stand + gift wrap + express  CRC 29,000.00
  USB-C hub + insurance  CRC 18,900.00
  Notebook  CRC 3,500.00
  Subtotal: CRC 51,400.00
  Tax:      CRC 6,682.00
  Shipping: CRC 3,750.00
  TOTAL:    CRC 61,832.00

Inventory: reserved 1 items
Payment: charged CRC 53,592.75 via SINPE Movil
----- INVOICE: Luis -----
  Mechanical keyboard + gift wrap + insurance  CRC 45,675.00
  Subtotal: CRC 45,675.00
  Tax:      CRC 5,937.75
  Shipping: CRC 1,980.00
  TOTAL:    CRC 53,592.75
```

### Prompt 3: Crear GiftWrap

#### Prompt enviado

CONTEXTO: En el paquete tecshop existe la interfaz CartProduct, con getPrice() y describe(). La clase CartItem implementa esa interfaz y conserva su comportamiento original.

TAREA: Crea un archivo nuevo llamado GiftWrap.java.

ESTRUCTURA: La clase pública GiftWrap implementa CartProduct. Tiene un atributo private final CartProduct item, recibido por constructor. getPrice() devuelve el precio del artículo más 1500. describe() devuelve la descripción del artículo seguida de  + gift wrap.

REGLAS: No modifiques archivos existentes. No agregues bibliotecas.

TERMINADO: Compila y ejecuta Main. Como esta clase todavía no se usa, la salida debe seguir siendo idéntica a la salida base. Mostrame únicamente GiftWrap.java completo.

#### Evaluación de la respuesta

Completar después de comprobar que el extra suma CRC 1,500, agrega la descripción indicada y no modifica el artículo original.

#### Salida obtenida

[Pegar `GiftWrap.java`.]

#### Resultado de compilación

```text
PS C:\TEC\2-Semestre\POO\tecshop> javac -encoding UTF-8 -d out *.java
PS C:\TEC\2-Semestre\POO\tecshop> java -cp out tecshop.Main
Inventory: reserved 3 items
Payment: charged CRC 61,832.00 via credit card
----- INVOICE: Ana -----
  Laptop stand + gift wrap + express  CRC 29,000.00
  USB-C hub + insurance  CRC 18,900.00
  Notebook  CRC 3,500.00
  Subtotal: CRC 51,400.00
  Tax:      CRC 6,682.00
  Shipping: CRC 3,750.00
  TOTAL:    CRC 61,832.00

Inventory: reserved 1 items
Payment: charged CRC 53,592.75 via SINPE Movil
----- INVOICE: Luis -----
  Mechanical keyboard + gift wrap + insurance  CRC 45,675.00
  Subtotal: CRC 45,675.00
  Tax:      CRC 5,937.75
  Shipping: CRC 1,980.00
  TOTAL:    CRC 53,592.75
```

### Prompt 4: Crear Insurance

#### Prompt enviado

CONTEXTO: En el paquete tecshop existe CartProduct, que declara getPrice() y describe(). También existe GiftWrap, que implementa CartProduct y recibe otro CartProduct por constructor.

TAREA: Crea un archivo nuevo llamado Insurance.java.

ESTRUCTURA: La clase pública Insurance implementa CartProduct. Tiene un atributo private final CartProduct item, recibido por constructor. getPrice() devuelve el precio del artículo recibido multiplicado por 1.05. describe() devuelve la descripción del artículo seguida de  + insurance.

REGLAS: No modifiques archivos existentes. No agregues bibliotecas.

TERMINADO: Compila y ejecuta Main. Como esta clase todavía no se usa, la salida debe seguir siendo idéntica a la salida base. Mostrame únicamente Insurance.java completo.

#### Evaluación de la respuesta

Completar después de comprobar que el seguro agrega el 5 % al precio que recibe y añade la descripción correcta.

#### Salida obtenida

[Pegar `Insurance.java`.]

#### Resultado de compilación

```text
PS C:\TEC\2-Semestre\POO\tecshop> javac -encoding UTF-8 -d out *.java
PS C:\TEC\2-Semestre\POO\tecshop> java -cp out tecshop.Main
Inventory: reserved 3 items
Payment: charged CRC 61,832.00 via credit card
----- INVOICE: Ana -----
  Laptop stand + gift wrap + express  CRC 29,000.00
  USB-C hub + insurance  CRC 18,900.00
  Notebook  CRC 3,500.00
  Subtotal: CRC 51,400.00
  Tax:      CRC 6,682.00
  Shipping: CRC 3,750.00
  TOTAL:    CRC 61,832.00

Inventory: reserved 1 items
Payment: charged CRC 53,592.75 via SINPE Movil
----- INVOICE: Luis -----
  Mechanical keyboard + gift wrap + insurance  CRC 45,675.00
  Subtotal: CRC 45,675.00
  Tax:      CRC 5,937.75
  Shipping: CRC 1,980.00
  TOTAL:    CRC 53,592.75
```

### Prompt 5: Crear Express

#### Prompt enviado

CONTEXTO: En el paquete tecshop existe la interfaz CartProduct, con getPrice() y describe(). Las clases GiftWrap e Insurance implementan la interfaz y reciben un CartProduct por constructor.

TAREA: Crea un archivo nuevo llamado Express.java.

ESTRUCTURA: La clase pública Express implementa CartProduct. Tiene un atributo private final CartProduct item, recibido por constructor. getPrice() devuelve el precio del artículo más 2500. describe() devuelve la descripción del artículo seguida de  + express.

REGLAS: No modifiques archivos existentes. No agregues bibliotecas.

TERMINADO: Compila y ejecuta Main. Como esta clase todavía no se usa, la salida debe seguir siendo idéntica a la salida base. Mostrame únicamente Express.java completo.

#### Evaluación de la respuesta

Completar después de comprobar que el extra suma CRC 2,500 y agrega la descripción correcta.

#### Salida obtenida

[Pegar `Express.java`.]

#### Resultado de compilación

```text
PS C:\TEC\2-Semestre\POO\tecshop> javac -encoding UTF-8 -d out *.java
PS C:\TEC\2-Semestre\POO\tecshop> java -cp out tecshop.Main
Inventory: reserved 3 items
Payment: charged CRC 61,832.00 via credit card
----- INVOICE: Ana -----
  Laptop stand + gift wrap + express  CRC 29,000.00
  USB-C hub + insurance  CRC 18,900.00
  Notebook  CRC 3,500.00
  Subtotal: CRC 51,400.00
  Tax:      CRC 6,682.00
  Shipping: CRC 3,750.00
  TOTAL:    CRC 61,832.00

Inventory: reserved 1 items
Payment: charged CRC 53,592.75 via SINPE Movil
----- INVOICE: Luis -----
  Mechanical keyboard + gift wrap + insurance  CRC 45,675.00
  Subtotal: CRC 45,675.00
  Tax:      CRC 5,937.75
  Shipping: CRC 1,980.00
  TOTAL:    CRC 53,592.75
```

### Prompt 6: Actualizar InvoicePrinter

#### Prompt enviado

CONTEXTO: En el paquete tecshop existe la interfaz CartProduct, con getPrice() y describe(). CartItem implementa esa interfaz. La clase Cart todavía devuelve sus artículos como List<CartItem>. InvoicePrinter imprime la descripción y el precio de cada artículo.

TAREA: Modifica únicamente InvoicePrinter.java para que trabaje con CartProduct al recorrer e imprimir los artículos de un carrito.

ESTRUCTURA: Conserva el formato actual de la factura, los textos, los espacios, los decimales, el subtotal y el resto de los cálculos. Usa los métodos describe() y getPrice() de la interfaz.

REGLAS: No modifiques ningún otro archivo. No cambies la salida del programa. No agregues bibliotecas.

TERMINADO: Compila y ejecuta Main. La salida debe coincidir exactamente con la salida base. Mostrame únicamente InvoicePrinter.java completo.

#### Evaluación de la respuesta

Completar después de comprobar que la factura conserva exactamente su formato y puede imprimir artículos mediante `CartProduct`.

#### Salida obtenida

[Pegar `InvoicePrinter.java`.]

#### Resultado de compilación

```text
PS C:\TEC\2-Semestre\POO\tecshop> javac -encoding UTF-8 -d out *.java
PS C:\TEC\2-Semestre\POO\tecshop> java -cp out tecshop.Main
Inventory: reserved 3 items
Payment: charged CRC 61,832.00 via credit card
----- INVOICE: Ana -----
  Laptop stand + gift wrap + express  CRC 29,000.00
  USB-C hub + insurance  CRC 18,900.00
  Notebook  CRC 3,500.00
  Subtotal: CRC 51,400.00
  Tax:      CRC 6,682.00
  Shipping: CRC 3,750.00
  TOTAL:    CRC 61,832.00

Inventory: reserved 1 items
Payment: charged CRC 53,592.75 via SINPE Movil
----- INVOICE: Luis -----
  Mechanical keyboard + gift wrap + insurance  CRC 45,675.00
  Subtotal: CRC 45,675.00
  Tax:      CRC 5,937.75
  Shipping: CRC 1,980.00
  TOTAL:    CRC 53,592.75
```

### Prompt 7: Cambiar Cart para aceptar CartProduct

#### Prompt enviado

CONTEXTO: En el paquete tecshop existe la interfaz CartProduct. CartItem implementa esa interfaz y InvoicePrinter ya puede imprimir artículos de tipo CartProduct. La clase Cart almacena una lista de artículos y ofrece métodos para agregarlos, obtenerlos y calcular el subtotal.

TAREA: Modifica únicamente Cart.java para que almacene List<CartProduct> en lugar de List<CartItem>.

ESTRUCTURA: Conserva los métodos públicos existentes, sus responsabilidades y el cálculo del subtotal. El método para agregar artículos debe aceptar CartProduct; el método para obtener artículos debe devolver List<CartProduct>. No cambies la lógica de los cálculos.

REGLAS: No modifiques ningún otro archivo. No agregues bibliotecas. Mantén la compatibilidad con los usos actuales de CartItem, que implementa CartProduct.

TERMINADO: Compila y ejecuta Main. La salida debe coincidir exactamente con la salida base. Mostrame únicamente Cart.java completo.

#### Evaluación de la respuesta

Completar después de comprobar que el carrito acepta la interfaz y que los artículos originales continúan funcionando.

#### Salida obtenida

[Pegar de `Cart.java`.]

#### Resultado de compilación

```text
PS C:\TEC\2-Semestre\POO\tecshop> javac -encoding UTF-8 -d out *.java
PS C:\TEC\2-Semestre\POO\tecshop> java -cp out tecshop.Main
Inventory: reserved 3 items
Payment: charged CRC 61,832.00 via credit card
----- INVOICE: Ana -----
  Laptop stand + gift wrap + express  CRC 29,000.00
  USB-C hub + insurance  CRC 18,900.00
  Notebook  CRC 3,500.00
  Subtotal: CRC 51,400.00
  Tax:      CRC 6,682.00
  Shipping: CRC 3,750.00
  TOTAL:    CRC 61,832.00

Inventory: reserved 1 items
Payment: charged CRC 53,592.75 via SINPE Movil
----- INVOICE: Luis -----
  Mechanical keyboard + gift wrap + insurance  CRC 45,675.00
  Subtotal: CRC 45,675.00
  Tax:      CRC 5,937.75
  Shipping: CRC 1,980.00
  TOTAL:    CRC 53,592.75
```

### Prompt 8: Usar los extras desde Main

#### Prompt enviado

CONTEXTO: En el paquete tecshop existe CartProduct, implementada por CartItem, GiftWrap, Insurance y Express. Cada extra recibe un CartProduct por constructor y calcula su propio precio y descripción. Cart acepta CartProduct. El constructor actual de CartItem todavía permite crear artículos con sus parámetros originales.

TAREA: Modifica únicamente Main.java para construir los carritos utilizando los nuevos extras en lugar de los indicadores booleanos de los artículos.

ESTRUCTURA: Conserva los mismos artículos, precios base, pesos, clientes, ciudades, proveedores y métodos de pago. Para Ana, aplica GiftWrap y después Express al Laptop stand; aplica Insurance al USB-C hub; deja el Notebook sin extras. Para Luis, aplica primero GiftWrap y después Insurance al Mechanical keyboard. Mantén el orden de aplicación indicado, porque afecta el cálculo del precio.

REGLAS: No modifiques ningún otro archivo. No cambies los datos de compra ni la salida esperada. No agregues bibliotecas.

TERMINADO: Compila y ejecuta Main. La salida debe coincidir exactamente con la salida base, incluyendo el orden de las descripciones y todos los montos. Mostrame únicamente Main.java completo.

#### Evaluación de la respuesta

Completar después de comprobar que los extras se pueden combinar en tiempo de ejecución y que los montos y las descripciones coinciden con la salida base.

#### Salida obtenida

[Pegar `Main.java`.]

#### Resultado de compilación

```text
PS C:\TEC\2-Semestre\POO\tecshop> javac -encoding UTF-8 -d out *.java
PS C:\TEC\2-Semestre\POO\tecshop> java -cp out tecshop.Main
Inventory: reserved 3 items
Payment: charged CRC 61,832.00 via credit card
----- INVOICE: Ana -----
  Laptop stand + gift wrap + express  CRC 29,000.00
  USB-C hub + insurance  CRC 18,900.00
  Notebook  CRC 3,500.00
  Subtotal: CRC 51,400.00
  Tax:      CRC 6,682.00
  Shipping: CRC 3,750.00
  TOTAL:    CRC 61,832.00

Inventory: reserved 1 items
Payment: charged CRC 53,592.75 via SINPE Movil
----- INVOICE: Luis -----
  Mechanical keyboard + gift wrap + insurance  CRC 45,675.00
  Subtotal: CRC 45,675.00
  Tax:      CRC 5,937.75
  Shipping: CRC 1,980.00
  TOTAL:    CRC 53,592.75
```

### Prompt 9: Crear un cuarto extra sin modificar clases existentes

#### Prompt enviado

CONTEXTO: En el paquete tecshop existe CartProduct, con getPrice() y describe(). Los extras GiftWrap, Insurance y Express implementan esa interfaz y reciben un CartProduct por constructor.

TAREA: Crea un archivo nuevo llamado Engraving.java para representar un servicio de grabado.

ESTRUCTURA: La clase pública Engraving implementa CartProduct. Tiene un atributo private final CartProduct item, recibido por constructor. getPrice() devuelve el precio del artículo recibido más 3000. describe() devuelve la descripción del artículo seguida de  + engraving.

REGLAS: No modifiques ningún archivo existente. No agregues bibliotecas. No uses esta clase en las compras actuales de Main.

TERMINADO: Compila y ejecuta Main. La salida debe seguir siendo idéntica a la salida base. Mostrame únicamente Engraving.java completo.

#### Evaluación de la respuesta

Completar después de comprobar que se agregó el nuevo extra sin modificar clases existentes y que la salida de las compras actuales no cambió.

#### Salida obtenida

[Pegar `Engraving.java`.]

#### Resultado de compilación

```text
PS C:\TEC\2-Semestre\POO\tecshop> javac -encoding UTF-8 -d out *.java
PS C:\TEC\2-Semestre\POO\tecshop> java -cp out tecshop.Main
Inventory: reserved 3 items
Payment: charged CRC 61,832.00 via credit card
----- INVOICE: Ana -----
  Laptop stand + gift wrap + express  CRC 29,000.00
  USB-C hub + insurance  CRC 18,900.00
  Notebook  CRC 3,500.00
  Subtotal: CRC 51,400.00
  Tax:      CRC 6,682.00
  Shipping: CRC 3,750.00
  TOTAL:    CRC 61,832.00

Inventory: reserved 1 items
Payment: charged CRC 53,592.75 via SINPE Movil
----- INVOICE: Luis -----
  Mechanical keyboard + gift wrap + insurance  CRC 45,675.00
  Subtotal: CRC 45,675.00
  Tax:      CRC 5,937.75
  Shipping: CRC 1,980.00
  TOTAL:    CRC 53,592.75
```

---

## Ronda 3: Checkout

### Prompt 1: Crear CheckoutFacade

#### Prompt enviado

CONTEXTO: En el paquete tecshop existen WebCheckout y KioskCheckout. Ambos realizan la misma secuencia general de compra: reservar los artículos con Inventory, obtener el subtotal del carrito, calcular el impuesto con TaxCalculator, cotizar el envío con ShippingService, calcular el total, cobrar mediante PaymentGateway e imprimir la factura con InvoicePrinter. Los dos clientes utilizan diferentes proveedores de envío y métodos de pago. Conserva las clases de servicios existentes y sus métodos actuales.

TAREA: Crea un archivo nuevo llamado CheckoutFacade.java que centralice esa secuencia.

ESTRUCTURA: La clase debe tener un método público checkout(Cart cart, String customer, String city, String provider, String paymentMethod). El método debe ejecutar la misma secuencia y utilizar los servicios existentes. Debe conservar el orden de las operaciones, los mensajes, el formato de la factura y los cálculos actuales.

REGLAS: No modifiques Inventory, TaxCalculator, ShippingService, PaymentGateway ni InvoicePrinter. No cambies los datos de compra ni agregues bibliotecas. No dupliques dentro de la nueva clase las implementaciones de los servicios.

TERMINADO: Mostrame únicamente CheckoutFacade.java completo. Si los nombres o constructores reales de los servicios no coinciden con el contexto, utiliza los que ya existen en el proyecto, sin inventar métodos nuevos.


#### Evaluación de la respuesta

Completar después de comprobar que la nueva clase coordina los servicios existentes en el mismo orden y que no vuelve a implementar sus responsabilidades.

#### Salida obtenida

[Pegar `CheckoutFacade.java`.]

#### Resultado de compilación

```text
PS C:\TEC\2-Semestre\POO\tecshop> javac -encoding UTF-8 -d out *.java
PS C:\TEC\2-Semestre\POO\tecshop> java -cp out tecshop.Main
Inventory: reserved 3 items
Payment: charged CRC 61,832.00 via credit card
----- INVOICE: Ana -----
  Laptop stand + gift wrap + express  CRC 29,000.00
  USB-C hub + insurance  CRC 18,900.00
  Notebook  CRC 3,500.00
  Subtotal: CRC 51,400.00
  Tax:      CRC 6,682.00
  Shipping: CRC 3,750.00
  TOTAL:    CRC 61,832.00

Inventory: reserved 1 items
Payment: charged CRC 53,592.75 via SINPE Movil
----- INVOICE: Luis -----
  Mechanical keyboard + gift wrap + insurance  CRC 45,675.00
  Subtotal: CRC 45,675.00
  Tax:      CRC 5,937.75
  Shipping: CRC 1,980.00
  TOTAL:    CRC 53,592.75
```

### Prompt 2: Actualizar WebCheckout

#### Prompt enviado

CONTEXTO: En el paquete tecshop existe CheckoutFacade.java, con el método público checkout(Cart cart, String customer, String city, String provider, String paymentMethod). WebCheckout realiza actualmente el proceso completo de compra para el canal web.

TAREA: Modifica únicamente WebCheckout.java para delegar el proceso de compra en CheckoutFacade.

ESTRUCTURA: Conserva la forma en que se utiliza WebCheckout desde el resto del programa. El canal web debe continuar usando el proveedor "correos" y el método de pago "credit card". El proceso de compra debe ejecutarse una sola vez mediante la nueva clase.

REGLAS: No modifiques ningún otro archivo. No cambies la salida, los datos ni los montos. No agregues bibliotecas.

TERMINADO: Compila y ejecuta Main. La salida debe coincidir exactamente con la salida base. Mostrame únicamente WebCheckout.java completo.

#### Evaluación de la respuesta

Completar después de verificar que el canal web delega en `CheckoutFacade` y conserva el proveedor y el método de pago originales.

#### Salida obtenida

[Pegar `WebCheckout.java`.]

#### Resultado de compilación

```text
PS C:\TEC\2-Semestre\POO\tecshop> javac -encoding UTF-8 -d out *.java
PS C:\TEC\2-Semestre\POO\tecshop> java -cp out tecshop.Main
Inventory: reserved 3 items
Payment: charged CRC 61,832.00 via credit card
----- INVOICE: Ana -----
  Laptop stand + gift wrap + express  CRC 29,000.00
  USB-C hub + insurance  CRC 18,900.00
  Notebook  CRC 3,500.00
  Subtotal: CRC 51,400.00
  Tax:      CRC 6,682.00
  Shipping: CRC 3,750.00
  TOTAL:    CRC 61,832.00

Inventory: reserved 1 items
Payment: charged CRC 53,592.75 via SINPE Movil
----- INVOICE: Luis -----
  Mechanical keyboard + gift wrap + insurance  CRC 45,675.00
  Subtotal: CRC 45,675.00
  Tax:      CRC 5,937.75
  Shipping: CRC 1,980.00
  TOTAL:    CRC 53,592.75
```

### Prompt 3: Actualizar KioskCheckout

#### Prompt enviado

CONTEXTO: En el paquete tecshop existe CheckoutFacade.java, con el método público checkout(Cart cart, String customer, String city, String provider, String paymentMethod). KioskCheckout realiza actualmente el proceso completo de compra para el kiosco. WebCheckout ya delega en la nueva clase.

TAREA: Modifica únicamente KioskCheckout.java para delegar el proceso de compra en CheckoutFacade.

ESTRUCTURA: Conserva la forma en que se utiliza KioskCheckout desde el resto del programa. El kiosco debe continuar usando el proveedor "local" y el método de pago "SINPE Movil". El proceso de compra debe ejecutarse una sola vez mediante la nueva clase.

REGLAS: No modifiques ningún otro archivo. No cambies la salida, los datos ni los montos. No agregues bibliotecas.

TERMINADO: Compila y ejecuta Main. La salida debe coincidir exactamente con la salida base. Mostrame únicamente KioskCheckout.java completo.

#### Evaluación de la respuesta

Completar después de comprobar que ambos clientes delegan en `CheckoutFacade`, conservan sus proveedores y métodos de pago y no duplican el proceso de compra.

#### Salida obtenida

[Pegar `KioskCheckout.java`.]

#### Resultado de compilación


```text
PS C:\TEC\2-Semestre\POO\tecshop> javac -encoding UTF-8 -d out *.java
PS C:\TEC\2-Semestre\POO\tecshop> java -cp out tecshop.Main
Inventory: reserved 3 items
Payment: charged CRC 61,832.00 via credit card
----- INVOICE: Ana -----
  Laptop stand + gift wrap + express  CRC 29,000.00
  USB-C hub + insurance  CRC 18,900.00
  Notebook  CRC 3,500.00
  Subtotal: CRC 51,400.00
  Tax:      CRC 6,682.00
  Shipping: CRC 3,750.00
  TOTAL:    CRC 61,832.00

Inventory: reserved 1 items
Payment: charged CRC 53,592.75 via SINPE Movil
----- INVOICE: Luis -----
  Mechanical keyboard + gift wrap + insurance  CRC 45,675.00
  Subtotal: CRC 45,675.00
  Tax:      CRC 5,937.75
  Shipping: CRC 1,980.00
  TOTAL:    CRC 53,592.75
```
