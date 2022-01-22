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

### AMDP Method Oluşturma Adımları

Geliştirmelerimizi ADT üzerinden oluşturuyoruz.

![image](https://user-images.githubusercontent.com/26427511/150639340-e0733e39-3ad6-4cc0-804e-a0ef6f3bb4e1.png)

![image](https://user-images.githubusercontent.com/26427511/150639645-7d3903b3-aba9-4806-af9d-ef9c828e02c0.png)

```abap
CLASS zaatan_amdp_ex01 DEFINITION
  PUBLIC
  FINAL
  CREATE PUBLIC .

  PUBLIC SECTION.

    INTERFACES if_amdp_marker_hdb.

    TYPES: BEGIN OF ty_basedat,
             carrid    TYPE spfli-carrid,
             carrname  TYPE scarr-carrname,
             currcode  TYPE scarr-currcode,
             connid    TYPE spfli-connid,
             countryfr TYPE spfli-countryfr,
             cityfrom  TYPE spfli-cityfrom,
             airpfrom  TYPE spfli-airpfrom,
             countryto TYPE spfli-countryto,
             cityto    TYPE spfli-cityto,
             airpto    TYPE spfli-airpto,
             price     TYPE sflight-price,
             currency  TYPE sflight-currency,
             planetype TYPE sflight-planetype,
             seatsmax  TYPE sflight-seatsmax,
             seatsocc  TYPE sflight-seatsocc,
           END OF ty_basedat,
           tt_basedat TYPE STANDARD TABLE OF ty_basedat WITH DEFAULT KEY.

    CLASS-METHODS:
      get_flight_dat
        IMPORTING
          VALUE(im_carrid)  TYPE spfli-carrid
          VALUE(im_fldate)  TYPE sflight-fldate
        EXPORTING
          VALUE(ev_basedat) TYPE tt_basedat.

  PROTECTED SECTION.
  PRIVATE SECTION.
ENDCLASS.


CLASS zaatan_amdp_ex01 IMPLEMENTATION.
  METHOD get_flight_dat BY DATABASE PROCEDURE FOR HDB LANGUAGE SQLSCRIPT
                         OPTIONS READ-ONLY
                         USING scarr spfli sflight.
    ev_basedat = SELECT scarr.carrid,
                        scarr.carrname,
                        scarr.currcode,
                        spfli.connid,
                        spfli.countryfr,
                        spfli.cityfrom,
                        spfli.airpfrom,
                        spfli.countryto,
                        spfli.cityto,
                        spfli.airpto,
                        sflight.price,
                        sflight.currency,
                        sflight.planetype,
                        sflight.seatsmax,
                        sflight.seatsocc
                        FROM sflight
                          INNER JOIN scarr ON scarr.carrid = sflight.carrid
                          INNER JOIN spfli ON spfli.carrid = sflight.carrid
                                          AND spfli.connid = sflight.connid
                          WHERE sflight.carrid = :im_carrid
                            AND sflight.fldate = :im_fldate;
  ENDMETHOD.
ENDCLASS.
```
### Program İçinde Çağrılması:
```abap
*&---------------------------------------------------------------------*
*& Program ZAATAN_AMDP
*&---------------------------------------------------------------------*
*&
*&---------------------------------------------------------------------*
PROGRAM zaatan_amdp.

CLASS amdp_demo DEFINITION.
  PUBLIC SECTION.
    METHODS:
      run_amdp_dat
        RETURNING
          VALUE(rt_basedat) TYPE zaatan_amdp_ex01=>tt_basedat,
      show_cte_dat
        IMPORTING
          !im_basedat TYPE zaatan_amdp_ex01=>tt_basedat.
ENDCLASS.
CLASS amdp_demo IMPLEMENTATION.
  METHOD run_amdp_dat.
    DATA: _carrid TYPE sflights-carrid VALUE 'LH',
          _fldate TYPE sflights-fldate VALUE '20210107'.

    cl_demo_input=>request( CHANGING field = _carrid ).
    cl_demo_input=>request( CHANGING field = _fldate ).

    zaatan_amdp_ex01=>get_flıght_dat(
        EXPORTING
            im_carrid = _carrid
            im_fldate = _fldate
        IMPORTING
             ev_basedat = rt_basedat ).

  ENDMETHOD.
  METHOD show_cte_dat.
    IF NOT im_basedat[] IS INITIAL.
      cl_demo_output=>display( im_basedat ).
    ENDIF.
  ENDMETHOD.
ENDCLASS.

START-OF-SELECTION.

  DATA(app) = NEW amdp_demo( ) .
  app->show_cte_dat(
    EXPORTING
     im_basedat = app->run_amdp_dat( ) ).
```
#### Çıktı;
![image](https://user-images.githubusercontent.com/26427511/150648681-630bc352-d6d4-49f1-be6d-6805059b1b2a.png)

