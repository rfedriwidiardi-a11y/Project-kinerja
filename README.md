# Project-kinerja
// Inisialisasi EmailJS
emailjs.init('FXF4s7JlMUYeHI6j0'); // ganti dengan public key EmailJS

let chart;

// Load data dari localStorage saat halaman dibuka
window.onload = function() {
  let data = localStorage.getItem('karyawanData');
  if(data){
    let arr = JSON.parse(data);
    arr.forEach(k => tambahBaris(k));
  }
  updateData(); // tampilkan grafik & notif awal
}

function tambahBaris(karyawan = null){
  let t=document.getElementById("tabel");
  let r=t.insertRow();
  for(let i=0;i<5;i++){
    let c=r.insertCell();
    if(i<4){
      let input = document.createElement("input");
      if(karyawan){
        if(i==0) input.value = karyawan.nama;
        if(i==1) input.value = karyawan.email;
        if(i==2) input.value = karyawan.target;
        if(i==3) input.value = karyawan.output;
      }
      c.appendChild(input);
    } else {
      c.innerText = karyawan ? karyawan.persen + "%" : "0%";
    }
  }
}

function updateData(){
  let t = document.getElementById("tabel");
  let labels = [], values = [];
  let notif = [];
  let karyawanArr = [];

  for(let i=1; i<t.rows.length; i++){
    let nama = t.rows[i].cells[0].children[0].value;
    let email = t.rows[i].cells[1].children[0].value;
    let target = parseFloat(t.rows[i].cells[2].children[0].value) || 0;
    let output = parseFloat(t.rows[i].cells[3].children[0].value) || 0;
    let persen = target ? ((output/target)*100).toFixed(1) : 0;

    t.rows[i].cells[4].innerText = persen + "%";
    labels.push(nama || "Karyawan " + i);
    values.push(persen);

    karyawanArr.push({nama, email, target, output, persen});

    if(persen <= 50 && email){
      notif.push(`⚠️ ${nama} (${email}) di bawah 50%`);

      // Kirim email otomatis via EmailJS
      emailjs.send('service_jfp4hw8', 'template_pgfl63e', {
        nama: nama,
        persen: persen,
        email: email
      }).then(function(response) {
        console.log('Email terkirim ke ' + email, response.status, response.text);
      }, function(error) {
        console.error('Gagal mengirim email ke ' + email, error);
      });
    }
  }

  // Simpan data ke localStorage
  localStorage.setItem('karyawanData', JSON.stringify(karyawanArr));

  tampilGrafik(labels, values);

  document.getElementById("notif").innerHTML =
    notif.length ?
    "<b>Notifikasi:</b><br>" + notif.join("<br>") :
    "<b>✅ Semua karyawan di atas 50%</b>";
}

function tampilGrafik(l,v){
  if(chart) chart.destroy();
  chart=new Chart(document.getElementById("grafik"),{
    type:"bar",
    data:{
      labels:l,
      datasets:[{
        label:"Persentase Kinerja",
        data:v,
        backgroundColor:"rgba(0,198,255,0.6)"
      }]
    },
    options:{
      scales:{
        y:{
          beginAtZero:true,
          max:100
        }
      }
    }
  });
}
