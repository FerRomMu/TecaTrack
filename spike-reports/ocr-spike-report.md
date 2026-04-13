# Evaluación de OCR para Comprobantes de Transferencia

## Objetivo

El sistema requiere procesar comprobantes bancarios para extraer campos estructurados (monto, fecha, identificadores, entidad, etc.) y cargarlos en la aplicación. 
El objetivo de este spike fue evaluar y comparar diferentes motores de OCR open source para extraer los datos de comprobantes en formato de imagen y PDF 

Para ambos formatos de comprobantes se probaron dos motores OCR: Tesseract OCR y PaddleOCR.


## Metodología de Prueba

Ambos enfoques compartieron un pipeline similar:
1. **Carga de Archivo:** Se convierten los PDFs a imágenes usando alta resolución (300 DPI) mediante `PyMuPDF` (`fitz`), o cargan las imagenes directamente mediante OpenCV/PIL.
2. **Preprocesamiento:** Se aplican filtros (`OpenCV`) para mejorar la legibilidad.
3. **Extracción de Texto (OCR):** Se pasa la imagen procesada por el motor OCR.
4. **Extracción de Campos:**  Se utilizan expresiones regulares para interceptar campos como el importe, CBU, CUIT/CUIL, banco y horario.



### Tesseract OCR
- **Dependencias:** Requirió instalar un binario externo a nivel sistema operativo `tesseract.exe` en Windows, además del wrapper de Python `pytesseract`. También, se debió agregar la ruta del binario al PATH. 
- **Preprocesamiento Aplicado:** Escala de grises y binarización adaptativa.
- **Extracción:** Proporciona un bloque de texto plano. Al carecer de entendimiento profundo del layout, la lectura de dos o más columnas tiende a mezclarse o perder el orden lógico.

### PaddleOCR
- **Dependencias:** Instalación 100% nativa vía `pip` (paquetes `paddlepaddle` y `paddleocr`). Descarga sus modelos de manera automática y no requiere instalar programas adicionales en el SO.
- **Preprocesamiento Aplicado:** Redimensionamiento y reducción de ruido.
- **Extracción:** Utiliza modelos avanzados basados en deep learning que devuelven no solo el texto, sino bounding boxes con coordenadas explícitas y nivel de *confianza* en la detección. Se configuró el reconocimiento considerando la orientación de las líneas de texto `use_textline_orientation=True`.


## Resultados y Comparación


| Característica | Tesseract OCR | PaddleOCR |
|---|---|---|
| **Instalación/Despliegue** | Complejo (Requiere binarios externos instalados en el SO) | Simple (Solo `pip install`, descarga modelos en runtime) |
| **Output** | Texto plano (string completo). | Estructurado (lista de diccionarios con coordenadas, texto y % de confianza). |
| **Precisión** | Requiere de mayor preprocesamiento para no fallar ante ruido o fondos. | Muy robusto. Logró detectar con mayor fidelidad montos, CUITs y CBUs sin modificar excesivamente la imagen original. |
| **Visualización y Debug** | Texto plano sobre consola. | Permite superponer los polígonos detectados sobre la imagen marcando en qué falló. |

Ambos OCR dieron buenos resultados para los comprobantes en formato PDF. Lo que mayormente marca la diferencia es en las imágenes.

Para las imágenes, Tesseract tuvo problemas para extraer y estructurar los campos. La mayoría de las palabras con tonos parecidos al fondo fueron parcial o directamente no reconocidos. Esto resultó en problemas de identificación posterior de los datos con las expresiones regulares. Para obtener alguna mejora se requería probar muchas variaciones de preprocesamiento de imagen.

Con PaddleOCR se obtuvo buenos resultados para las imágenes sin necesidad de tunning de las imágenes durante el preprocesamiento, logrando extraer exitosamente los datos más importantes como:

- Importe 
- Hora 
- CBU 
- CUIT/CUIL
- Entidad Financiera Origen

El único inconveniente fue que, específicamente para Windows, hay un bug en las compatibilidad de `paddlepaddle` y `paddleocr` para las versiones 3.x por lo que las pruebas se realizaron sobre la version 2.9.2


## Conclusión 

Basado en los resultados y la experiencia de desarrollo, la herramienta ganadora para el proyecto es PaddleOCR por:

1. **Precisión Superior:** Demostró ser más preciso y entiende mejor el layout espacial de las capturas. 
2. **Estructura Computable:** Al retornar `Bounding Boxes` además de texto, es posible en un futuro mapear datos no solo por regex, sino por coordenadas relativas.
3. **Despliegue Facilitado:** Al depender enteramente de Python, previene la necesidad de configurar infraestructuras Docker o la necesidad de instalaciones de software adicionales


