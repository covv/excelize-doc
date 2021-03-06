# Escritura en streaming

StreamWriter definió el tipo de escritor de flujo.

```go
type StreamWriter struct {
    File    *File
    Sheet   string
    SheetID int
    // contiene campos filtrados o no exportados
}
```

Cell se puede utilizar directamente en StreamWriter.SetRow para especificar un estilo y un valor.

```go
type Cell struct {
    StyleID int
    Formula string
    Value   interface{}
}
```

## Obtener escritor de flujo {#NewStreamWriter}

```go
func (f *File) NewStreamWriter(sheet string) (*StreamWriter, error)
```

NewStreamWriter devuelve la estructura del escritor de flujo por el nombre de la hoja de trabajo para generar una nueva hoja de trabajo con grandes cantidades de datos. Tenga en cuenta que después de establecer filas, debe llamar al método [`Flush`](stream.md#Flush) para finalizar el proceso de escritura de transmisión y asegurarse de que el orden de los números de línea sea ascendente, la API común y la API de flujo no se pueden combinar para escribir datos en las hojas de trabajo. Por ejemplo, configure los datos para la hoja de trabajo de tamaño `102400` filas x `50` columnas con números y estilo:

```go
file := excelize.NewFile()
streamWriter, err := file.NewStreamWriter("Sheet1")
if err != nil {
    fmt.Println(err)
}
styleID, err := file.NewStyle(`{"font":{"color":"#777777"}}`)
if err != nil {
    fmt.Println(err)
}
if err := streamWriter.SetRow("A1", []interface{}{
    excelize.Cell{StyleID: styleID, Value: "Data"}}); err != nil {
    fmt.Println(err)
}
for rowID := 2; rowID <= 102400; rowID++ {
    row := make([]interface{}, 50)
    for colID := 0; colID < 50; colID++ {
        row[colID] = rand.Intn(640000)
    }
    cell, _ := excelize.CoordinatesToCellName(1, rowID)
    if err := streamWriter.SetRow(cell, row); err != nil {
        fmt.Println(err)
    }
}
if err := streamWriter.Flush(); err != nil {
    fmt.Println(err)
}
if err := file.SaveAs("Book1.xlsx"); err != nil {
    fmt.Println(err)
}
```

Establezca el valor de celda y la fórmula de celda para una hoja de trabajo con el escritor de flujo:

```go
err := streamWriter.SetRow("A1", []interface{}{
    excelize.Cell{Value: 1},
    excelize.Cell{Value: 2},
    excelize.Cell{Formula: "SUM(A1,B1)"}});
```

## Escribir fila de hoja para transmitir {#SetRow}

```go
func (sw *StreamWriter) SetRow(axis string, slice interface{}) error
```

SetRow escribe una matriz en la fila de flujo por el nombre de la hoja de trabajo, la coordenada inicial y un puntero al tipo de matriz `slice`. Tenga en cuenta que debe llamar al método [`Flush`](stream.md#Flush) para finalizar el proceso de escritura de transmisión.

## Corriente de vaciado {#Flush}

```go
func (sw *StreamWriter) Flush() error
```

Flush finalizando el proceso de escritura en streaming.
