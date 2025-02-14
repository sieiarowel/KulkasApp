# KulkasApp
Aplikasi pengelolaan data kulkas berbasis Java dan MySQL.  

# Fitur
Menambahkan data kulkas  
Mengedit data kulkas  
Menghapus data kulkas  
Mencari kulkas berdasarkan merek  
Tampilan berbasis GUI dengan Swing  

# Kode : 
```
import javax.swing.*;
import javax.swing.table.DefaultTableModel;
import java.awt.*;
import java.awt.event.*;
import java.sql.*;

public class KulkasApp extends JFrame {
    private JTextField txtMerek, txtKapasitas, txtTahun;
    private JRadioButton rb1Pintu, rb2Pintu;
    private JButton btnTambah, btnEdit, btnHapus, btnCari, btnRefresh;
    private JTable table;
    private DefaultTableModel model;
    private Connection conn;
    
    public KulkasApp() {
        setTitle("Pengelolaan Data Kulkas Caroline Tangka");
        setSize(600, 400);
        setDefaultCloseOperation(EXIT_ON_CLOSE);
        setLocationRelativeTo(null);
        initUI();
        connectDB();
        loadData();
    }
    
    private void initUI() {
        setLayout(new BorderLayout());
        
        
        JPanel panelInput = new JPanel(new GridLayout(5, 2));
        panelInput.add(new JLabel("Merek: "));
        txtMerek = new JTextField();
        panelInput.add(txtMerek);

        panelInput.add(new JLabel("Jenis: "));
        JPanel panelJenis = new JPanel();
        rb1Pintu = new JRadioButton("1 Pintu");
        rb2Pintu = new JRadioButton("2 Pintu");
        ButtonGroup bgJenis = new ButtonGroup();
        bgJenis.add(rb1Pintu);
        bgJenis.add(rb2Pintu);
        panelJenis.add(rb1Pintu);
        panelJenis.add(rb2Pintu);
        panelInput.add(panelJenis);

        panelInput.add(new JLabel("Kapasitas (Liter): "));
        txtKapasitas = new JTextField();
        panelInput.add(txtKapasitas);

        panelInput.add(new JLabel("Tahun Produksi: "));
        txtTahun = new JTextField();
        panelInput.add(txtTahun);
        
        add(panelInput, BorderLayout.NORTH);


        model = new DefaultTableModel(new String[]{"ID", "Merek", "Jenis", "Kapasitas", "Tahun"}, 0);
        table = new JTable(model);
        add(new JScrollPane(table), BorderLayout.CENTER);
        
    
        JPanel panelButton = new JPanel();
        btnTambah = new JButton("Tambah");
        btnEdit = new JButton("Edit");
        btnHapus = new JButton("Hapus");
        btnCari = new JButton("Cari");
        btnRefresh = new JButton("Refresh");
        panelButton.add(btnTambah);
        panelButton.add(btnEdit);
        panelButton.add(btnHapus);
        panelButton.add(btnCari);
        panelButton.add(btnRefresh);
        add(panelButton, BorderLayout.SOUTH);
        
        btnTambah.addActionListener(e -> tambahData());
        btnEdit.addActionListener(e -> editData());
        btnHapus.addActionListener(e -> hapusData());
        btnCari.addActionListener(e -> cariData());
        btnRefresh.addActionListener(e -> loadData());
    }
    
    private void connectDB() {
        try {
        
            Class.forName("com.mysql.cj.jdbc.Driver");

            
            String url = "jdbc:mysql://localhost:3306/db_kulkas?serverTimezone=UTC";
            String user = "root"; 
            String password = ""; 

            conn = DriverManager.getConnection(url, user, password);
            System.out.println("Database terhubung!");
        } catch (ClassNotFoundException e) {
            JOptionPane.showMessageDialog(this, "Driver MySQL tidak ditemukan!", "Error", JOptionPane.ERROR_MESSAGE);
            e.printStackTrace();
        } catch (SQLException e) {
            JOptionPane.showMessageDialog(this, "Gagal terhubung ke database!\n" + e.getMessage(), "Error", JOptionPane.ERROR_MESSAGE);
            e.printStackTrace();
        }
    }
    
    private void loadData() {
        model.setRowCount(0);
        try {
            if (conn == null) {
                connectDB();
            }
            
            Statement stmt = conn.createStatement();
            ResultSet rs = stmt.executeQuery("SELECT * FROM kulkas");

            while (rs.next()) {
                model.addRow(new Object[]{
                    rs.getInt("id_kulkas"), 
                    rs.getString("merek"), 
                    rs.getString("jenis"),
                    rs.getInt("kapasitas"), 
                    rs.getInt("tahun_produksi")
                });
            }
            
            rs.close();
            stmt.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
    
    private void tambahData() {
        try {
            String merek = txtMerek.getText();
            String jenis = rb1Pintu.isSelected() ? "1 Pintu" : "2 Pintu";
            int kapasitas = Integer.parseInt(txtKapasitas.getText());
            int tahun = Integer.parseInt(txtTahun.getText());

            PreparedStatement ps = conn.prepareStatement("INSERT INTO kulkas (merek, jenis, kapasitas, tahun_produksi) VALUES (?, ?, ?, ?)");
            ps.setString(1, merek);
            ps.setString(2, jenis);
            ps.setInt(3, kapasitas);
            ps.setInt(4, tahun);
            ps.executeUpdate();

            JOptionPane.showMessageDialog(this, "Data berhasil ditambahkan!");
            loadData();
        } catch (Exception e) {
            e.printStackTrace();
            JOptionPane.showMessageDialog(this, "Gagal menambah data!");
        }
    }
    
    private void editData() {
        int selectedRow = table.getSelectedRow();
        if (selectedRow == -1) {
            JOptionPane.showMessageDialog(this, "Pilih data yang ingin diedit!", "Error", JOptionPane.ERROR_MESSAGE);
            return;
        }
        
        int idKulkas = (int) model.getValueAt(selectedRow, 0);
        String merek = txtMerek.getText();
        String jenis = rb1Pintu.isSelected() ? "1 Pintu" : "2 Pintu";
        int kapasitas = Integer.parseInt(txtKapasitas.getText());
        int tahun = Integer.parseInt(txtTahun.getText());

        try {
            String sql = "UPDATE kulkas SET merek = ?, jenis = ?, kapasitas = ?, tahun_produksi = ? WHERE id_kulkas = ?";
            PreparedStatement ps = conn.prepareStatement(sql);
            ps.setString(1, merek);
            ps.setString(2, jenis);
            ps.setInt(3, kapasitas);
            ps.setInt(4, tahun);
            ps.setInt(5, idKulkas);
            
            ps.executeUpdate();
            JOptionPane.showMessageDialog(this, "Data berhasil diperbarui!");
            loadData();
        } catch (SQLException e) {
            JOptionPane.showMessageDialog(this, "Gagal memperbarui data!", "Error", JOptionPane.ERROR_MESSAGE);
            e.printStackTrace();
        }
    }
    
    private void hapusData() {
        int selectedRow = table.getSelectedRow();
        if (selectedRow == -1) {
            JOptionPane.showMessageDialog(this, "Pilih data yang ingin dihapus!", "Error", JOptionPane.ERROR_MESSAGE);
            return;
        }
        
        int idKulkas = (int) model.getValueAt(selectedRow, 0);
        
        int confirm = JOptionPane.showConfirmDialog(this, "Apakah Anda yakin ingin menghapus data?", "Konfirmasi", JOptionPane.YES_NO_OPTION);
        if (confirm == JOptionPane.YES_OPTION) {
            try {
                String sql = "DELETE FROM kulkas WHERE id_kulkas = ?";
                PreparedStatement ps = conn.prepareStatement(sql);
                ps.setInt(1, idKulkas);
                ps.executeUpdate();
                
                JOptionPane.showMessageDialog(this, "Data berhasil dihapus!");
                loadData();
            } catch (SQLException e) {
                JOptionPane.showMessageDialog(this, "Gagal menghapus data!", "Error", JOptionPane.ERROR_MESSAGE);
                e.printStackTrace();
            }
        }
    }
    
    private void cariData() {
        String keyword = txtMerek.getText().toLowerCase();
        model.setRowCount(0);
        
        try {
            String sql = "SELECT * FROM kulkas WHERE merek LIKE ?";
            PreparedStatement ps = conn.prepareStatement(sql);
            ps.setString(1, "%" + keyword + "%");
            ResultSet rs = ps.executeQuery();

            while (rs.next()) {
                model.addRow(new Object[]{
                    rs.getInt("id_kulkas"),
                    rs.getString("merek"),
                    rs.getString("jenis"),
                    rs.getInt("kapasitas"),
                    rs.getInt("tahun_produksi")
                });
            }

            rs.close();
            ps.close();
        } catch (SQLException e) {
            JOptionPane.showMessageDialog(this, "Gagal melakukan pencarian!", "Error", JOptionPane.ERROR_MESSAGE);
            e.printStackTrace();
        }
    }
    
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> new KulkasApp().setVisible(true));
    }
}
