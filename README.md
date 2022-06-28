# butce_hesaplama
from flask import Flask, render_template, request, redirect, url_for
import sqlite3

def kullaniciveriekle(id, kullaniciadi, sifre):
    global veri
    with sqlite3.connect("kullanici.db") as con:
        cur=con.cursor()
        cur.execute("insert into tblkullanici (id, kullaniciadi, sifre) values (?, ?, ?)", (id, kullaniciadi, sifre))
        #cur.execute("insert into tblgelir (kullaniciid) select id from tblkullanici")
        #cur.execute("insert into tblgider (kullaniciid) select id from tblkullanici")
        con.commit()


def gelirveriekle(tarih, kategori, miktar, kullaniciid):
    with sqlite3.connect("kullanici.db") as con:
        cur=con.cursor()
        cur.execute("insert into tblgelir (tarih, kategori, miktar, kullaniciid) values (?, ?, ?, ?)", (tarih, kategori, miktar, kullaniciid))
        con.commit()

def giderveriekle(tarih, kategori, miktar,kullaniciid):
    with sqlite3.connect("kullanici.db") as con:
        cur=con.cursor()
        cur.execute("insert into tblgider (tarih, kategori, miktar, kullaniciid) values (?, ?, ?, ?)", (tarih, kategori, miktar, kullaniciid))
        con.commit()

datagelir = []
def gelirveriAl():
    global datagelir
    with sqlite3.connect("kullanici.db") as con:
        cur = con.cursor()
        cur.execute("select * from tblgelir order by id desc ")
        datagelir = cur.fetchall()

datagider=[]
def giderveriAl():
    global datagider
    with sqlite3.connect("kullanici.db") as con:
        cur = con.cursor()
        cur.execute("select * from tblgider order by id desc ")
        datagider = cur.fetchall()

datakullanici=[]
def kullaniciveriAl():
    global datakullanici, veri
    with sqlite3.connect("kullanici.db") as con:
        cur = con.cursor()
        cur.execute("select * from tblkullanici order by id desc ")
        datakullanici = cur.fetchall()

def gelirveriSil(id):
    with sqlite3.connect("kullanici.db") as con:
        cur=con.cursor()
        cur.execute("delete from tblgelir where id=?",(id,))

def giderveriSil(id):
    with sqlite3.connect("kullanici.db") as con:
        cur=con.cursor()
        cur.execute("delete from tblgider where id=?",(id,))

toplamGelir=0
def toplamgelir():
    global toplamGelir
    with sqlite3.connect("kullanici.db") as con:
        cur = con.cursor()
        cur.execute("select sum(miktar) from tblgelir")
        toplamGelir=cur.fetchall()

toplamGider=0
def toplamgider():
    global toplamGider
    with sqlite3.connect("kullanici.db") as con:
        cur = con.cursor()
        cur.execute("select sum(miktar) from tblgider")
        toplamGider=cur.fetchall()

kullaniciveriAl()
giderveriAl()
gelirveriAl()
app=Flask(__name__)

@app.route("/")
def hesapla():
    toplamgider()
    toplamgelir()
    for i in toplamGelir:
        gelir=i[0]
    for a in toplamGider:
        gider=a[0]
        butce=gelir-gider
    return render_template("index.html", butce=butce)

@app.route("/giris", methods=["POST", "GET"])
def giris():
    if request.method=="POST":
        id=request.form["id"]
        kullaniciadi=request.form["kullaniciadi"]
        sifre=request.form["sifre"]
        kullaniciveriekle(id, kullaniciadi, sifre)
        kullaniciveriAl()
    return render_template("giris.html", datakullanici=datakullanici)
    

@app.route("/gdr", methods=["POST", "GET"])
def gdr():
    if request.method=="POST":
        tarih=request.form["tarih"]
        kategori=request.form["kategori"]
        miktar=request.form["miktar"]
        kullaniciid=request.form["kullaniciid"]
        giderveriekle(tarih, kategori, miktar, kullaniciid)
    toplamgider()
    giderveriAl()
    return render_template("gdr.html", toplamGider=toplamGider, datagider=datagider)

@app.route("/gelir", methods=["POST", "GET"])
def gelir():
    gelirveriAl()
    toplamgelir()
    if request.method=="POST":
        tarih=request.form["tarih"]
        kategori=request.form["kategori"]
        miktar=request.form["miktar"]
        kullaniciid=request.form["kullaniciid"]
        gelirveriekle(tarih, kategori, miktar, kullaniciid)
    return render_template("gelir.html", toplamGelir=toplamGelir, datagelir=datagelir)

@app.route("/gelirdelete/<string:id>")
def gelirdelete(id):
    gelirveriSil(id)
    return redirect(url_for("gelir"))

@app.route("/giderdelete/<string:id>")
def giderdelete(id):
    giderveriSil(id)
    return redirect(url_for("gdr"))


if __name__=="__main__":
    app.run(debug=True)
