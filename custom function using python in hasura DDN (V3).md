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
