# Custom Business Logic in Hasura DDN

source: https://hasura.io/docs/3.0/getting-started/build/add-business-logic

Pada kesempatan kali ini saya akan mencoba mempelajari tentang custom logic yang menjadi fitur baru dari hasura versi 3 yang sebelumnya belum ada di versi 2. sebelum lebih lanjut, custom business logic pada hasura v2 bisa dilakukan dengan menggunakan fitur action yang akan memanggil http API endpoint dari suatu service lain untuk diterapkan menjadi custom logic dan dapat digunakan menjadi query di hasura.

Terdapat kekurangan yang ada dari hal ini yaitu diperlukannya menginstance custom logic tersebut menjadi sebuah service lain untuk kemudian baru dapat dipanggil dan digunakan di hasura.

Pada hasura DDN / V3, custom logic dapat diterapkan secara langsung tanpa perlu membuat service lain yang mana akan diinstance bersamaan dengan hasura itu sendiri. Untuk custom logic saat ini support untuk dapat ditulis pada 3 bahasa pemrograman yaitu typescript, python dan go.

![image](https://github.com/user-attachments/assets/255c1024-7711-4d11-8e1c-b4718f2142f3)

custom logic membutuhkan lambda connector untuk dapat terhubung ke hasura sesuai dengan bahasa yang digunakan.

## Praktek

Kali ini saya akan mencoba membuat custom logic dengan menggunakan python untuk melakukan konversi nilai waktu agar dapat lebih mudah dibaca / readable.

Terdapat beberapa ketentuan yang diperlukan sebelum lanjut yaitu antara lain:
* Sudah memiliki DDN CLI, VS Code extension, dan Docker yang terinstall
* project DDN baru atau yang sudah ada dan setidaknya memiliki 1 subgraph
* Python version >=3.11

### inisialisasi lambda connector

Pertama masuk ke direktori project DDN, lalu buka terminal untuk menjalankan DDN CLI dengan command: `ddn connector init -i`, pilih `hasura/python` sebagai jenis connectornya, berikan nama connectornya (disini saya memberi nama `my_python`), pilih portnya (tekan enter saja jika ingin dibuatkan secara otomatis). Ini akan menginisialisasi connector baru yang ada di folder `sub_graph/connector/my_python`.

Buka file `functions.py` yang sudah di generate secara otomatis, disinilah tempat penulisan custom logic dilakukan. Secara default, dalam file tersebut sudah terdapat sejumlah custom logic, abaikan dan tambahkan function baru berikut:

```
from datetime import datetime

connector = FunctionConnector()

@connector.register_query
async def format_timezone_date(date_string: str) -> str:
    if "." in date_string:
        date_part, millisecond_part = date_string.split(".")
        millisecond_part = millisecond_part.ljust(6, "0")
        date_string = f"{date_part}.{millisecond_part}"

    try:
        date = datetime.fromisoformat(date_string)
    except ValueError:
        return "Invalid date format"

    day = date.day
    nth = lambda d: "th" if 11 <= d <= 13 else {1: "st", 2: "nd", 3: "rd"}.get(d % 10, "th")

    hours = date.hour
    ampm = "pm" if hours >= 12 else "am"
    hour = hours % 12 or 12

    day_of_week = date.strftime("%A")
    month = date.strftime("%B")
    year = date.year

    return f"{hour}{ampm}, {day_of_week}, {month} {day}{nth(day)}, {year}."
```

simpan.

### introspect connector

setelah custom function ditambahkan, lakukan introspect dengan command `ddn connector introspect my_python` untuk maping functionnya menjadi file metadata berformat .hml.

### Tambahkan command dari custom function

Agar command setiap custom function yang ada dapat di gunakan dalam query dan dihitung sebagai asset, lakukan command `ddn command add my_python "*"` untuk maping setiap command.

### Build ke local

setelah di introspect dan di tambahkan command nya, lakukan build supergraph ke local agar dapat langsung diterapkan di graphql local, berikut command nya: `ddn supergraph build local`

### Mencobanya di local

jalankan command `ddn run docker-start` untuk memulai graphql engine di local via docker dan buka consolenya dengan command `ddn console --local`

Setelah console terbuka, query dapat langsung dijalankan seperti berikut:

![image](https://github.com/user-attachments/assets/1c80f450-b415-46c1-8fa9-4e5f09a1a43d)

## Membuat relasi

source: https://hasura.io/docs/3.0/getting-started/build/create-a-relationship

salah satu relationship yang dapat dilakukan yaitu dari object type seperti kolom di database dengan custom model ataupun command. pada kasus kali ini saya akan membuat relasi antara suatu kolom di tabel database postgre yang berformat timestamp, dengan command `formatTimezoneDate` yang sudah saya buat diatas sehingga nantinya akan muncul entitas baru hasil relasi antar keduanya yang dapat langsung mengkonversi format timestamp dari database menjadi format yang mudah dibaca/readable.

Sebelum lebih lanjut, terdapat syarat yang diperlukan untuk melakukan relationship yaitu hasura ddn juga sudah terhubung dengan sebuah database (dalam hal ini database postgre) melalui connector yang lain selain dari connector custom logic python.

### Edit metadata hml dari tabel database

pada folder metadata, setiap tabel dari database akan menjadi satu file .hml baru, masuk ke tabel yang ingin direlasikan (disini saya masuk ke file Matchsatujuta.hml dari tabel Matchsatujuta) lalu ke paling bawah dan mulai mengetikan `---` lalu enter, karena sudah memiliki vscode extention hasura, maka akan langsung direkomendasikan beberapa optionnya, pilih relationship dan nanti akan langsung di generate formatnya, isi bagian yang kosong sesuai dengan isian berikut:

```
---
kind: Relationship
version: v1
definition:
  name: readableLandingTimestamp
  sourceType: Matchsatujuta //tabelnya
  target:
    command:
      name: FormatTimezoneDate //nama command dari custom logicnya
  mapping:
    - source:
        fieldPath:
          - fieldName: landingTimestamp //kolom tabel yang ingin direlasikan
      target:
        argument:
          argumentName: dateString //target untuk dijadikan param pada command custom logic

---
kind: Relationship
version: v1
definition:
  name: readableKafkaTimestamp
  sourceType: Matchsatujuta 
  target:
    command:
      name: FormatTimezoneDate
  mapping:
    - source:
        fieldPath:
          - fieldName: kafkaTimestamp
      target:
        argument:
          argumentName: dateString
```

Simpan

### Build ke local

lakukan build supergraph ke local agar dapat langsung diterapkan di graphql local, berikut command nya: `ddn supergraph build local`

### Mencobanya di local

jalankan command `ddn run docker-start` untuk memulai graphql engine di local via docker dan buka consolenya dengan command `ddn console --local`

Setelah console terbuka, query dapat langsung dijalankan seperti berikut:

![image](https://github.com/user-attachments/assets/fa8d5bb3-1b2e-46f6-a69b-7c0e0422ec73)

Dapat dilihat terdapat field tambahan berdasarkan relasi yang dibuat yang jika dipanggil di query graphql akan melakukan konversi secara langsung timestamp dari tabel di database menjadi format yang lebih readable mengikuti custom function yang sudah di buat dari python.

Berikut adalah contoh custom logic lain yang sudah dibuat menggunakan python menggunakan connector lambda python:

![image](https://github.com/user-attachments/assets/f9b9e2de-d05e-4292-b8a6-2721aafb4b03)

source code `functions.py`
```
from hasura_ndc import start
from hasura_ndc.instrumentation import with_active_span # If you aren't planning on adding additional tracing spans, you don't need this!
from opentelemetry.trace import get_tracer # If you aren't planning on adding additional tracing spans, you don't need this either!
from hasura_ndc.function_connector import FunctionConnector
from pydantic import BaseModel # You only need this import if you plan to have complex inputs/outputs, which function similar to how frameworks like FastAPI do
import asyncio # You might not need this import if you aren't doing asynchronous work
from hasura_ndc.errors import UnprocessableContent
from datetime import datetime # Don't forget to import datetime at the top of the file!

connector = FunctionConnector()

@connector.register_query
async def format_timezone_date(date_string: str) -> str:
    # Cek apakah timestamp memiliki mili-detik kurang dari 6 digit
    if "." in date_string:
        # Pisahkan tanggal dan mili-detik
        date_part, millisecond_part = date_string.split(".")
        # Lengkapi mili-detik hingga 6 digit dengan menambahkan '0'
        millisecond_part = millisecond_part.ljust(6, "0")
        # Gabungkan kembali menjadi satu string
        date_string = f"{date_part}.{millisecond_part}"
    
    try:
        # Parse timestamp dengan format ISO
        date = datetime.fromisoformat(date_string)
    except ValueError:
        return "Invalid date format"

    day = date.day
    nth = lambda d: "th" if 11 <= d <= 13 else {1: "st", 2: "nd", 3: "rd"}.get(d % 10, "th")

    hours = date.hour
    ampm = "pm" if hours >= 12 else "am"
    hour = hours % 12 or 12

    day_of_week = date.strftime("%A")
    month = date.strftime("%B")
    year = date.year

    return f"{hour}{ampm}, {day_of_week}, {month} {day}{nth(day)}, {year}."


# This is an example of a simple function that can be added onto the graph
@connector.register_query # This is how you register a query
def hello(name: str) -> str:
    return f"Hello {name}"

# You can use Nullable parameters, but they must default to None
# The FunctionConnector also doesn't care if your functions are sync or async, so use whichever you need!
@connector.register_query
async def nullable_hello(name: str | None = None) -> str:
    return f"Hello {name if name is not None else 'world'}"

# Parameters that are untyped accept any scalar type, arrays, or null and are treated as JSON.
# Untyped responses or responses with indeterminate types are treated as JSON as well!
@connector.register_mutation # This is how you register a mutation
def some_mutation_function(any_type_param):
    return any_type_param

# Similar to frameworks like FastAPI, you can use Pydantic Models for inputs and outputs
class Pet(BaseModel):
    name: str
    
class Person(BaseModel):
    name: str
    pets: list[Pet] | None = None

@connector.register_query
def greet_person(person: Person) -> str:
    greeting = f"Hello {person.name}!"
    if person.pets is not None:
        for pet in person.pets:
            greeting += f" And hello to {pet.name}.."
    else:
        greeting += f" I see you don't have any pets."
    return greeting

class ComplexType(BaseModel):
    lists: list[list] # This turns into a List of List's of any valid JSON!
    person: Person | None = None # This becomes a nullable attribute that accepts a person type from above
    x: int # You can also use integers
    y: float # As well as floats
    z: bool # And booleans

# When the outputs are typed with Pydantic models you can select which attributes you want returned!
@connector.register_query
def complex_function(input: ComplexType) -> ComplexType:
    return input

# This last section shows you how to add Otel tracing to any of your functions!
tracer = get_tracer("ndc-sdk-python.server") # You only need a tracer if you plan to add additional Otel spans

# Utilizing with_active_span allows the programmer to add Otel tracing spans
@connector.register_query
async def with_tracing(name: str) -> str:

    def do_some_more_work(_span, work_response):
        return f"Hello {name}, {work_response}"

    async def the_async_work_to_do():
        # This isn't actually async work, but it could be! Perhaps a network call belongs here, the power is in your hands fellow programmer!
        return "That was a lot of work we did!"

    async def do_some_async_work(_span):
        work_response = await the_async_work_to_do()
        return await with_active_span(
            tracer,
            "Sync Work Span",
            lambda span: do_some_more_work(span, work_response), # Spans can wrap synchronous functions, and they can be nested for fine-grained tracing
            {"attr": "sync work attribute"}
        )

    return await with_active_span(
        tracer,
        "Root Span that does some async work",
        do_some_async_work, # Spans can wrap asynchronous functions
        {"tracing-attr": "Additional attributes can be added to Otel spans by making use of with_active_span like this"}
    )

# This is an example of how to setup queries to be run in parallel
@connector.register_query(parallel_degree=5) # When joining to this function, it will be executed in parallel in batches of 5
async def parallel_query(name: str) -> str:
    await asyncio.sleep(1)
    return f"Hello {name}"

# This is an example of how you can throw an error
# There are different error types including: BadRequest, Forbidden, Conflict, UnprocessableContent, InternalServerError, NotSupported, and BadGateway
@connector.register_query
def error():
    raise UnprocessableContent(message="This is a error", details={"Error": "This is a error!"})

if __name__ == "__main__":
    start(connector)
```
