# ABAP Managed Database Procedures(AMDP)

  Veritabanı Prosedürü, veritabanında saklanan ve yürütülen bir işlevdir. ABAP ABAP Managed Database Procedures olarak adlandırılan AMDP, temel olarak
ABAP ortamında veritabanı prosedürü yazmak için kullanılır. Bir AMDP sınıfı, normal global sınıflarla aynı şekilde oluşturulur, tek farkı IF_AMDP_MARKER_HDB arayüzüne sahip olmasıdır.IF_AMDP_MARKER_HDB arayüzünü uygulamak, sınıfa yeni yeni bir yöntem eklemez sadece sınıfın artık bir AMDP sınıfı olarak tanımlanmasını sağlar. 

  Çoğu yeni geliştirme nesnesi gibi, AMDP sınıfları da yalnızca ADT'den (Eclipse) üzerinden düzenlenebilir. 
  
### AMDP Genel Özellikleri
  Bir AMDP methotları, statik methotlar veya instance olarak tanımladığımız methodlar olabilir. AMDP sınıflarının sıradan yöntemlerden hiçbir farkı yoktur. Bunun bir AMDP yöntemi olduğunu ancak kaynak koduna bakarak anlayabilirsiniz. 

#### AMDP yöntemleri bazı kısıtlamalara sahiptir;

##### Arayüz Kısıtlamaları;
- Genel türleri (TYPE DATA, TYPE CSEQUENCE, CLIKE vb.) kullanılmaz, yalnızca temel veri türleri ve tablo veri türleri kullanılabilir.
- Parametreler referans olarak tanımlanamaz.
- RETURNING parametrelerine izin verilmez.
- Yalnızca giriş parametreleri isteğe bağlı olarak işaretlenebilir ve tüm isteğe bağlı parametreler bir başlangıç değerinin verilmesini gerekmektedir.
- f, decfloat16, decfloat34, string ve xstring türündeki parametreler varsayılan bir değere sahip olamaz, yani isteğe bağlı olamazlar.
- CHANGING parametreleri string veya xstring türünde olamaz.
- C veya N tipi parametrelerin uzunluğu 5000 karakteri aşamaz.
- İstisnaları tanımlarken, yalnızca önceden tanımlanmış istisna sınıfları kullanılabilir . Klasik istisnalar mevcut değildir.
- Parametre adları: %_ öneki ile başlayamaz.
- Bağlantı adlı bir parametre kullanılıyorsa, - DBCON_NAME türünde olmalıdır.

##### Uygulama Kısıtlamaları;
- DDL sözdizimini kullanamazsınız.
- AMDP yöntemi boş olamaz.
- Yapıcı bir AMDP yöntemi olarak uygulanamaz.
- Select-option parametreleri ile ABAP programında cl_shdb_seltab=>combine_seltabs metodunu kullanarak string değerine dönüştürmeli AMDP methodu içinde filtrelemek için APPLY_FILTER ifadesi ile filitrelemeler kullananılabilir.
- AMDP ile MSEG gibi tabloları kullanamıyoruz. Bunun yerine MATDOC tablosunu veya proxy nesnesini (NSDM_V_MSEG) kullanabiliriz.

### AMDP Kullanımı
```abap
METHOD <meth> BY DATABASE PROCEDURE 
              FOR <db>
              LANGUAGE <db_lang>
              [OPTIONS <db_options>]
    [USING <db_entities>].

< Insert SQLScript code here >

ENDMETHOD.
```

### AMDP Oluşturma Adımları

Geliştirmelerimizi ADT üzerinden oluşturuyoruz.


![image](https://user-images.githubusercontent.com/26427511/150639340-e0733e39-3ad6-4cc0-804e-a0ef6f3bb4e1.png)

![image](https://user-images.githubusercontent.com/26427511/150639645-7d3903b3-aba9-4806-af9d-ef9c828e02c0.png)





