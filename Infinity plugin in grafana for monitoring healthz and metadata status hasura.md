# Latar belakang

Sebagai bentuk observability, berbagai keadaan diperlukan untuk di monitor dari instance hasura, termasuk dari pada health dari berbagai sumbernya, seperti kesehatan koneksi dari database, remote schema, actions, bahkan health dari hasura sendiri. 

Untuk mengecek kesehatan koneksi database, dari hasura sendiri sudah tersedia di versi v2 nya di dokumentasi ini: https://hasura.io/docs/2.0/deployment/health-checks/source-health-check/

source health check dapat langsung diterapkan untuk kemudian akan diperiksa secara berkala dan di eksport dalam bentuk metric `hasura_source_health`.

berbeda dengan database, sumber lainnya seperti remote schema tidak dapat dilakukan hal yang serupa karena belum support. untuk itu, maka jika terdapat gangguan koneksi atau health nya tidak sehat baik itu timeout, dsb akan mengakibatkan INCONSISTENT METADATA. untuk itu maka untuk mengecek status metadatanya secara teratur dan dapat dimonitoring, diperlukan plugin dari Infinity dari grafana yang berguna untuk mengakses endpoint API [`get_inconsistency_metadata`](https://hasura.io/docs/2.0/api-reference/schema-metadata-api/manage-metadata/#schema-metadata-get-inconsistent-metadata) untuk mengambil status metadata dan detailnya jika terjadinya inkonsistensi metadata.

Selain itu, infinity juga dapat digunakan untuk mengecek endpoint /healthz dari hasura untuk mengecek kesehatan hasura secara keseluruhan sebagai pengganti penggunaan service blackbox yang ada dalam template dashboard grafana hasura health dari hasura.

referensi: https://www.youtube.com/watch?v=IoToI15bgY8

#langkah-langkah

untuk dokumentasi infinity plugin secara detail dapat dilihat disini: https://grafana.com/grafana/plugins/yesoreyeram-infinity-datasource/?tab=overview

jika grafana dapat terhubung ke internet maka instalasi dapat dilakukan langsung dengan masuk ke administration->plugins lalu tinggal search infinity lalu install dan jadikan data source.

untuk kasus grafana yang tidak dapat mencapai internet, maka plugin perlu di download terlebih dahulu dalam bentuk zip [disini](https://grafana.com/grafana/plugins/yesoreyeram-infinity-datasource/?tab=installation). import ke server tempat grafana di install lalu unzip dan letakan di folder `var/lib/grafana/plugins` dan restart service grafana nya.

jika instalasi sudah berhasil, jadikan infinity sebagai datasource dan langsung dapat digunakan.

Untuk memantau sebuah endpoint nya, buat dashboard dan buat visualisasi baru. jadikan infinity sebagai source data nya

## endpoint get inconsistency metadata

untuk endpoint get inconsistent metadata, terdapat 2 kolom yaitu is_consistent berbentuk boolean dan inconsistent_objects berbentuk array/tabel.

### is_consistent

rename query name nya menjadi `is_consistent` , pilih type `JSON`, parser `backend` , format `data frame`, masukan url instance hasura + `/v1/query` dengan method `POST`. Untuk body content pilih type json dan masukan body content 

`{
    "type": "get_inconsistent_metadata",
    "args": {}
}`

masukan juga header x-hasura-admin-secret dan valuenya, lalu parsing rows/root nya isi `is_consistent`

![image](https://github.com/user-attachments/assets/facc419f-a497-4b27-a9a2-05887c4cdbdd)

kemudian untuk di sisi kanan, pilih jenis visualisasi stat lalu di bagian value option -> fields dan pilih nama query `is_consistent`, scroll kebawah lagi dan masukan value mapingnya sesuai yang diinginkan seperti contoh dibawah

![image](https://github.com/user-attachments/assets/d32187ae-a93f-4bbc-9a2a-d23e2c262bf2)

### inconsistent_objects

untuk field inconsistent_objects, lakukan tahap yang sama namun rename query namenya `inconsistent_objects`,  masukan parsing rows/root nya ke `inconsistent_objects` dan untuk bentuk visualisasinya pilih tabel , di bagian value option -> fields dan pilih nama query `is_consistent`.

## endpoint healthz

untuk visualisasi healthz status, buat visualisasi baru, pilih infinity sebagai data sourcenya dan rename query name nya sebagai `healthz`.

pilih type `HTML`, parser `backend` , format `data frame`, masukan url instance hasura + `/healthz` dengan method `GET` tanpa perlu menambahkan header dan request body.

![image](https://github.com/user-attachments/assets/a987e147-a66d-4da7-a279-5730832ef2a6)

di sisi kanan di bagian value option -> fields dan pilih nama query `healthz`.

jika sudah berhasil semua maka akan menampilkan visual berikut jika metadata consistent

![image](https://github.com/user-attachments/assets/b6c8d6c0-e53a-449c-878b-e21c3e09d194)

