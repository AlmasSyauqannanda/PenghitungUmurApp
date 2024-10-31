
# Aplikasi Perhitungan Umur

Latihan 2: Aplikasi Penghitung Usia

Aplikasi Penghitung Usia adalah proyek latihan dalam modul pembelajaran, yang dirancang untuk menentukan usia seseorang berdasarkan tanggal lahir yang dipilih melalui antarmuka GUI berbasis Java. Selain menghitung usia, aplikasi ini juga menampilkan peristiwa-peristiwa bersejarah yang terjadi pada hari kelahiran tersebut, memberikan informasi tambahan yang menarik bagi pengguna.

# Deskripsi
Aplikasi ini dibangun menggunakan JFrame untuk menyediakan antarmuka yang mudah digunakan. Melalui dateChooser, pengguna dapat memilih tanggal lahir mereka, dan aplikasi akan secara otomatis menghitung serta menampilkan usia dalam tahun, bulan, dan hari. Selain itu, aplikasi ini terhubung dengan API eksternal yang menyajikan berbagai peristiwa penting yang terjadi pada tanggal ulang tahun pengguna, menjadikannya pengalaman yang tidak hanya informatif tetapi juga menambah wawasan.

# Features
Fitur Utama

Penghitungan Usia Akurat: Aplikasi ini menghitung usia pengguna secara rinci, meliputi tahun, bulan, dan hari untuk hasil yang tepat.

Ulang Tahun Mendatang: Menampilkan tanggal serta hari ulang tahun pengguna yang berikutnya, berdasarkan tanggal lahir yang telah diinput.

Peristiwa Bersejarah: Memberikan informasi tentang kejadian-kejadian bersejarah yang terjadi pada tanggal yang sama, diambil langsung dari API online.

Aplikasi ini dibangun dengan memanfaatkan komponen Java Swing sebagai antarmuka, menggunakan LocalDate dan ZoneId dari Java Time API untuk mengelola perhitungan tanggal. HttpURLConnection digunakan untuk mengambil data dari internet, sementara pustaka JSON dipakai untuk parsing data dari API, memudahkan aplikasi dalam mengolah dan menampilkan informasi kepada pengguna.




## PenghitunganUmurHelper

    import java.time.LocalDate;
    import java.time.Period;
    import java.io.BufferedReader;
    import java.io.InputStreamReader;
    import java.net.HttpURLConnection;
    import java.net.URL;
    import java.util.function.Supplier;
    import javax.swing.JTextArea;
    import org.json.JSONArray;
    import org.json.JSONObject;


    public class PenghitungUmurHelper {
        // Menghitung umur secara detail (tahun, bulan, hari)

        public String hitungUmurDetail(LocalDate lahir, LocalDate sekarang) {
            Period period = Period.between(lahir, sekarang);
            return period.getYears() + " tahun, " + period.getMonths() + " bulan, " + period.getDays() + " hari";
        }

    // Menghitung hari ulang tahun berikutnya
        public LocalDate hariUlangTahunBerikutnya(LocalDate lahir, LocalDate sekarang) {
            LocalDate ulangTahunBerikutnya = lahir.withYear(sekarang.getYear());
            if (!ulangTahunBerikutnya.isAfter(sekarang)) {
                ulangTahunBerikutnya = ulangTahunBerikutnya.plusYears(1);
            }
            return ulangTahunBerikutnya;
        }

    // Menerjemahkan teks hari ke bahasa Indonesia
        public String getDayOfWeekInIndonesian(LocalDate date) {
            switch (date.getDayOfWeek()) {
                case MONDAY:
                    return "Senin";
                case TUESDAY:
                    return "Selasa";
                case WEDNESDAY:
                    return "Rabu";
                case THURSDAY:
                    return "Kamis";
                case FRIDAY:
                    return "Jumat";
                case SATURDAY:
                    return "Sabtu";
                case SUNDAY:
                    return "Minggu";
                default:
                    return "";
            }
        }
    // Mendapatkan peristiwa penting secara baris per baris

        public void getPeristiwaBarisPerBaris(LocalDate tanggal, JTextArea txtAreaPeristiwa, Supplier<Boolean> shouldStop) {
            try {
                // Periksa jika thread seharusnya dihentikan sebelum dimulai
                if (shouldStop.get()) {
                    return;
                }

                String urlString = "https://byabbe.se/on-this-day/"
                        + tanggal.getMonthValue() + "/" + tanggal.getDayOfMonth() + "/events.json";
                URL url = new URL(urlString);
                HttpURLConnection conn = (HttpURLConnection) url.openConnection();
                conn.setRequestMethod("GET");
                conn.setRequestProperty("User-Agent", "Mozilla/5.0");

                int responseCode = conn.getResponseCode();
                if (responseCode != 200) {
                    throw new Exception("HTTP response code: " + responseCode
                            + ". Silakan coba lagi nanti atau cek koneksi internet.");
                }

                BufferedReader in = new BufferedReader(new InputStreamReader(conn.getInputStream()));
                String inputLine;
                StringBuilder content = new StringBuilder();

                while ((inputLine = in.readLine()) != null) {
                    // Periksa jika thread seharusnya dihentikan saat membaca data
                    if (shouldStop.get()) {
                        in.close();
                        conn.disconnect();
                        javax.swing.SwingUtilities.invokeLater(()
                                -> txtAreaPeristiwa.setText("Pengambilan data dibatalkan.\n"));
                        return;
                    }
                    content.append(inputLine);
                }

                in.close();
                conn.disconnect();

                JSONObject json = new JSONObject(content.toString());
                JSONArray events = json.getJSONArray("events");

                for (int i = 0; i < events.length(); i++) {
                    // Periksa jika thread seharusnya dihentikan sebelum memproses data
                    if (shouldStop.get()) {
                        javax.swing.SwingUtilities.invokeLater(()
                                -> txtAreaPeristiwa.setText("Pengambilan data dibatalkan.\n"));
                        return;
                    }
                    JSONObject event = events.getJSONObject(i);
                    String year = event.getString("year");
                    String description = event.getString("description");
                    String translatedDescription = translateToIndonesian(description);
                    String peristiwa = year + ": " + translatedDescription;
                    javax.swing.SwingUtilities.invokeLater(()
                            -> txtAreaPeristiwa.append(peristiwa + "\n"));
                }

                if (events.length() == 0) {
                    javax.swing.SwingUtilities.invokeLater(()
                            -> txtAreaPeristiwa.setText("Tidak ada peristiwa penting yang ditemukan pada tanggal ini."));
                }
            } catch (Exception e) {
                javax.swing.SwingUtilities.invokeLater(()
                        -> txtAreaPeristiwa.setText("Gagal mendapatkan data peristiwa: " + e.getMessage()));
            }
        }
        // Menerjemahkan teks ke bahasa Indonesia

        private String translateToIndonesian(String text) {
            try {
                String urlString = "https://lingva.ml/api/v1/en/id/"
                        + text.replace(" ", "%20");
                URL url = new URL(urlString);
                HttpURLConnection conn = (HttpURLConnection) url.openConnection();
                conn.setRequestMethod("GET");
                conn.setRequestProperty("User-Agent", "Mozilla/5.0");
                int responseCode = conn.getResponseCode();
                if (responseCode != 200) {
                    throw new Exception("HTTP response code: " + responseCode);
                }
                BufferedReader in = new BufferedReader(new InputStreamReader(conn.getInputStream(), "utf-8"));
                String inputLine;
                StringBuilder content = new StringBuilder();
                while ((inputLine = in.readLine()) != null) {
                    content.append(inputLine);
                }
                in.close();
                conn.disconnect();
                JSONObject json = new JSONObject(content.toString());
                return json.getString("translation");
            } catch (Exception e) {
                return text + " (Gagal diterjemahkan)";
            }
        }
    }

## PenghitunganUmurFrame
    import java.time.LocalDate;
    import java.time.ZoneId;
    import java.time.format.DateTimeFormatter;
    import java.util.Date;
    public class PenghitunganUmurFrame extends javax.swing.JFrame {
        private PenghitungUmurHelper helper;
        private volatile boolean stopFetching = false;
        private Thread peristiwaThread;
        public PenghitunganUmurFrame() {
            initComponents();
            helper = new PenghitungUmurHelper();
        }

        /**
        * This method is called from within the constructor to initialize the form.
        * WARNING: Do NOT modify this code. The content of this method is always
        * regenerated by the Form Editor.
        */
        @SuppressWarnings("unchecked")
        // <editor-fold defaultstate="collapsed" desc="Generated Code">                          
        private void initComponents() {

            jPanel1 = new javax.swing.JPanel();
            jLabel1 = new javax.swing.JLabel();
            jLabel2 = new javax.swing.JLabel();
            jLabel3 = new javax.swing.JLabel();
            dateChooserTanggalLahir = new com.toedter.calendar.JDateChooser();
            txtUmur = new javax.swing.JTextField();
            txtHariUlangTahunBerikutnya = new javax.swing.JTextField();
            HitungUmur = new javax.swing.JButton();
            Keluar = new javax.swing.JButton();
            jPanel2 = new javax.swing.JPanel();
            jScrollPane1 = new javax.swing.JScrollPane();
            txtAreaPeristiwa = new javax.swing.JTextArea();

            setDefaultCloseOperation(javax.swing.WindowConstants.EXIT_ON_CLOSE);

            jLabel1.setText("Pilih Tanggal Lahir");

            jLabel2.setText("Umur Anda");

            jLabel3.setText("Hari Ulang Tahun Berikutnya ");

            dateChooserTanggalLahir.addPropertyChangeListener(new java.beans.PropertyChangeListener() {
                public void propertyChange(java.beans.PropertyChangeEvent evt) {
                    dateChooserTanggalLahirPropertyChange(evt);
                }
            });

            txtUmur.addActionListener(new java.awt.event.ActionListener() {
                public void actionPerformed(java.awt.event.ActionEvent evt) {
                    txtUmurActionPerformed(evt);
                }
            });

            HitungUmur.setText("Hitung");
            HitungUmur.addActionListener(new java.awt.event.ActionListener() {
                public void actionPerformed(java.awt.event.ActionEvent evt) {
                    HitungUmurActionPerformed(evt);
                }
            });

            Keluar.setText("Keluar");
            Keluar.addActionListener(new java.awt.event.ActionListener() {
                public void actionPerformed(java.awt.event.ActionEvent evt) {
                    KeluarActionPerformed(evt);
                }
            });

            jPanel2.setLayout(new java.awt.GridLayout());

            txtAreaPeristiwa.setColumns(20);
            txtAreaPeristiwa.setRows(20);
            jScrollPane1.setViewportView(txtAreaPeristiwa);

            javax.swing.GroupLayout jPanel1Layout = new javax.swing.GroupLayout(jPanel1);
            jPanel1.setLayout(jPanel1Layout);
            jPanel1Layout.setHorizontalGroup(
                jPanel1Layout.createParallelGroup(javax.swing.GroupLayout.Alignment.LEADING)
                .addGroup(jPanel1Layout.createSequentialGroup()
                    .addGap(22, 22, 22)
                    .addGroup(jPanel1Layout.createParallelGroup(javax.swing.GroupLayout.Alignment.LEADING)
                        .addGroup(jPanel1Layout.createSequentialGroup()
                            .addComponent(jScrollPane1, javax.swing.GroupLayout.PREFERRED_SIZE, 555, javax.swing.GroupLayout.PREFERRED_SIZE)
                            .addPreferredGap(javax.swing.LayoutStyle.ComponentPlacement.RELATED)
                            .addComponent(jPanel2, javax.swing.GroupLayout.DEFAULT_SIZE, javax.swing.GroupLayout.DEFAULT_SIZE, Short.MAX_VALUE))
                        .addGroup(jPanel1Layout.createSequentialGroup()
                            .addGroup(jPanel1Layout.createParallelGroup(javax.swing.GroupLayout.Alignment.LEADING)
                                .addComponent(jLabel1)
                                .addComponent(jLabel2)
                                .addComponent(jLabel3))
                            .addGap(30, 30, 30)
                            .addGroup(jPanel1Layout.createParallelGroup(javax.swing.GroupLayout.Alignment.LEADING, false)
                                .addGroup(jPanel1Layout.createSequentialGroup()
                                    .addComponent(HitungUmur)
                                    .addPreferredGap(javax.swing.LayoutStyle.ComponentPlacement.RELATED, 31, Short.MAX_VALUE)
                                    .addComponent(Keluar))
                                .addComponent(txtUmur)
                                .addComponent(txtHariUlangTahunBerikutnya)
                                .addComponent(dateChooserTanggalLahir, javax.swing.GroupLayout.DEFAULT_SIZE, javax.swing.GroupLayout.DEFAULT_SIZE, Short.MAX_VALUE))
                            .addGap(0, 0, Short.MAX_VALUE)))
                    .addContainerGap())
            );
            jPanel1Layout.setVerticalGroup(
                jPanel1Layout.createParallelGroup(javax.swing.GroupLayout.Alignment.LEADING)
                .addGroup(jPanel1Layout.createSequentialGroup()
                    .addContainerGap()
                    .addGroup(jPanel1Layout.createParallelGroup(javax.swing.GroupLayout.Alignment.TRAILING)
                        .addGroup(jPanel1Layout.createSequentialGroup()
                            .addGroup(jPanel1Layout.createParallelGroup(javax.swing.GroupLayout.Alignment.LEADING)
                                .addComponent(jLabel1)
                                .addComponent(dateChooserTanggalLahir, javax.swing.GroupLayout.PREFERRED_SIZE, javax.swing.GroupLayout.DEFAULT_SIZE, javax.swing.GroupLayout.PREFERRED_SIZE))
                            .addGap(18, 18, 18)
                            .addComponent(jLabel2))
                        .addComponent(txtUmur, javax.swing.GroupLayout.PREFERRED_SIZE, javax.swing.GroupLayout.DEFAULT_SIZE, javax.swing.GroupLayout.PREFERRED_SIZE))
                    .addGap(18, 18, 18)
                    .addGroup(jPanel1Layout.createParallelGroup(javax.swing.GroupLayout.Alignment.BASELINE)
                        .addComponent(jLabel3, javax.swing.GroupLayout.PREFERRED_SIZE, 16, javax.swing.GroupLayout.PREFERRED_SIZE)
                        .addComponent(txtHariUlangTahunBerikutnya, javax.swing.GroupLayout.PREFERRED_SIZE, javax.swing.GroupLayout.DEFAULT_SIZE, javax.swing.GroupLayout.PREFERRED_SIZE))
                    .addGap(18, 18, 18)
                    .addGroup(jPanel1Layout.createParallelGroup(javax.swing.GroupLayout.Alignment.BASELINE)
                        .addComponent(HitungUmur)
                        .addComponent(Keluar))
                    .addGap(26, 26, 26)
                    .addGroup(jPanel1Layout.createParallelGroup(javax.swing.GroupLayout.Alignment.LEADING, false)
                        .addComponent(jPanel2, javax.swing.GroupLayout.DEFAULT_SIZE, 223, Short.MAX_VALUE)
                        .addComponent(jScrollPane1, javax.swing.GroupLayout.PREFERRED_SIZE, 0, Short.MAX_VALUE))
                    .addContainerGap(javax.swing.GroupLayout.DEFAULT_SIZE, Short.MAX_VALUE))
            );

            javax.swing.GroupLayout layout = new javax.swing.GroupLayout(getContentPane());
            getContentPane().setLayout(layout);
            layout.setHorizontalGroup(
                layout.createParallelGroup(javax.swing.GroupLayout.Alignment.LEADING)
                .addGroup(layout.createSequentialGroup()
                    .addContainerGap()
                    .addComponent(jPanel1, javax.swing.GroupLayout.PREFERRED_SIZE, javax.swing.GroupLayout.DEFAULT_SIZE, javax.swing.GroupLayout.PREFERRED_SIZE)
                    .addContainerGap(javax.swing.GroupLayout.DEFAULT_SIZE, Short.MAX_VALUE))
            );
            layout.setVerticalGroup(
                layout.createParallelGroup(javax.swing.GroupLayout.Alignment.LEADING)
                .addGroup(layout.createSequentialGroup()
                    .addContainerGap()
                    .addComponent(jPanel1, javax.swing.GroupLayout.DEFAULT_SIZE, javax.swing.GroupLayout.DEFAULT_SIZE, Short.MAX_VALUE)
                    .addContainerGap())
            );

            pack();
        }// </editor-fold>                        

        private void txtUmurActionPerformed(java.awt.event.ActionEvent evt) {                                        
            // TODO add your handling code here:
        }                                       

        private void HitungUmurActionPerformed(java.awt.event.ActionEvent evt) {                                           
        Date tanggalLahir = dateChooserTanggalLahir.getDate();
        if (tanggalLahir != null) {
            // Menghitung umur dan hari ulang tahun berikutnya
            LocalDate lahir = tanggalLahir.toInstant().atZone(ZoneId.systemDefault()).toLocalDate();
            LocalDate sekarang = LocalDate.now();
            
            // Menghitung umur
            String umur = helper.hitungUmurDetail(lahir, sekarang);
            txtUmur.setText(umur);

            // Menghitung tanggal ulang tahun berikutnya
            LocalDate ulangTahunBerikutnya = helper.hariUlangTahunBerikutnya(lahir, sekarang);
            String hariUlangTahunBerikutnya = helper.getDayOfWeekInIndonesian(ulangTahunBerikutnya);

            // Format tanggal ulang tahun berikutnya
            DateTimeFormatter formatter = DateTimeFormatter.ofPattern("dd-MM-yyyy");
            String tanggalUlangTahunBerikutnya = ulangTahunBerikutnya.format(formatter);
            
            // Menampilkan hasil pada field txtHariUlangTahunBerikutnya
            txtHariUlangTahunBerikutnya.setText(hariUlangTahunBerikutnya + " (" + tanggalUlangTahunBerikutnya + ")");
            
            // Set stop flag untuk thread sebelumnya
        stopFetching = true;
        if (peristiwaThread != null && peristiwaThread.isAlive()) {
            peristiwaThread.interrupt(); // Beri sinyal ke thread untuk berhenti
        }

        // Reset flag untuk thread baru
        stopFetching = false;

        // Mendapatkan peristiwa penting secara asinkron
        peristiwaThread = new Thread(() -> {
            try {
                txtAreaPeristiwa.setText("Tunggu, sedang mengambil data...\n");
                helper.getPeristiwaBarisPerBaris(ulangTahunBerikutnya, txtAreaPeristiwa, () -> stopFetching);
                
                if (!stopFetching) {
                    javax.swing.SwingUtilities.invokeLater(() -> 
                        txtAreaPeristiwa.append("Selesai mengambil data peristiwa"));
                }
            } catch (Exception e) {
                if (Thread.currentThread().isInterrupted()) {
                    javax.swing.SwingUtilities.invokeLater(() -> 
                        txtAreaPeristiwa.setText("Pengambilan data dibatalkan.\n"));
                }
            }
        });
        }

        
        
        peristiwaThread.start();

        }                                          

        private void KeluarActionPerformed(java.awt.event.ActionEvent evt) {                                       
            System.exit(0);
        }                                      

        private void dateChooserTanggalLahirPropertyChange(java.beans.PropertyChangeEvent evt) {                                                       
        txtUmur.setText("");
        txtHariUlangTahunBerikutnya.setText("");
        // Hentikan thread yang sedang berjalan saat tanggal lahir berubah
            stopFetching = true;
            if (peristiwaThread != null && peristiwaThread.isAlive()) {
            peristiwaThread.interrupt();
            }
            txtAreaPeristiwa.setText("");
        }                                                      

        /**
        * @param args the command line arguments
        */
        public static void main(String args[]) {
            /* Set the Nimbus look and feel */
            //<editor-fold defaultstate="collapsed" desc=" Look and feel setting code (optional) ">
            /* If Nimbus (introduced in Java SE 6) is not available, stay with the default look and feel.
            * For details see http://download.oracle.com/javase/tutorial/uiswing/lookandfeel/plaf.html 
            */
            try {
                for (javax.swing.UIManager.LookAndFeelInfo info : javax.swing.UIManager.getInstalledLookAndFeels()) {
                    if ("Nimbus".equals(info.getName())) {
                        javax.swing.UIManager.setLookAndFeel(info.getClassName());
                        break;
                    }
                }
            } catch (ClassNotFoundException ex) {
                java.util.logging.Logger.getLogger(PenghitunganUmurFrame.class.getName()).log(java.util.logging.Level.SEVERE, null, ex);
            } catch (InstantiationException ex) {
                java.util.logging.Logger.getLogger(PenghitunganUmurFrame.class.getName()).log(java.util.logging.Level.SEVERE, null, ex);
            } catch (IllegalAccessException ex) {
                java.util.logging.Logger.getLogger(PenghitunganUmurFrame.class.getName()).log(java.util.logging.Level.SEVERE, null, ex);
            } catch (javax.swing.UnsupportedLookAndFeelException ex) {
                java.util.logging.Logger.getLogger(PenghitunganUmurFrame.class.getName()).log(java.util.logging.Level.SEVERE, null, ex);
            }
            //</editor-fold>

            /* Create and display the form */
            java.awt.EventQueue.invokeLater(new Runnable() {
                public void run() {
                    new PenghitunganUmurFrame().setVisible(true);
                }
            });
        }

        // Variables declaration - do not modify                     
        private javax.swing.JButton HitungUmur;
        private javax.swing.JButton Keluar;
        private com.toedter.calendar.JDateChooser dateChooserTanggalLahir;
        private javax.swing.JLabel jLabel1;
        private javax.swing.JLabel jLabel2;
        private javax.swing.JLabel jLabel3;
        private javax.swing.JPanel jPanel1;
        private javax.swing.JPanel jPanel2;
        private javax.swing.JScrollPane jScrollPane1;
        private javax.swing.JTextArea txtAreaPeristiwa;
        private javax.swing.JTextField txtHariUlangTahunBerikutnya;
        private javax.swing.JTextField txtUmur;
        // End of variables declaration                   
    }








## Authors

- [@Almas Syauqannanda]

