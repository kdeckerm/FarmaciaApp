//main

package pedidofarmaciaapp;
import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.util.Date;
import javax.swing.SpinnerDateModel;

public class PedidoFarmaciaApp {

    public static void main(String[] args) {
        // Ejecuta la GUI en un hilo de eventos de Swing
        SwingUtilities.invokeLater(() -> new PedidoFarmaciaGUI());
    }
}

// Clase para la interfaz gráfica del pedido

class PedidoFarmaciaGUI extends JFrame {
    private JTextField txtMedicamento, txtCantidad;
    private JComboBox<String> cmbTipo;
    private JRadioButton rbCofarma, rbEmpsephar, rbCemefar;
    private JCheckBox chkPrincipal, chkSecundaria;
    private JButton btnConfirmar, btnBorrar;
    private JSpinner spinnerFecha;
    
    public PedidoFarmaciaGUI() {
        setTitle("Pedido de Medicamentos");
        setSize(600, 450);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setLayout(new GridBagLayout());

        // Inicialización de componentes
        txtMedicamento = new JTextField(15);
        txtCantidad = new JTextField(5);
        String[] tipos = {"Analgésico", "Analéptico", "Anestésico", "Antiácido", "Antidepresivo", "Antibiótico"};
        cmbTipo = new JComboBox<>(tipos);

        rbCofarma = new JRadioButton("Cofarma");
        rbEmpsephar = new JRadioButton("Empsephar");
        rbCemefar = new JRadioButton("Cemefar");
        ButtonGroup grupoDistribuidores = new ButtonGroup();
        grupoDistribuidores.add(rbCofarma);
        grupoDistribuidores.add(rbEmpsephar);
        grupoDistribuidores.add(rbCemefar);

        chkPrincipal = new JCheckBox("Principal");
        chkSecundaria = new JCheckBox("Secundaria");

        // Selector de fecha
        spinnerFecha = new JSpinner(new SpinnerDateModel());
        JSpinner.DateEditor editor = new JSpinner.DateEditor(spinnerFecha, "dd/MM/yyyy");
        spinnerFecha.setEditor(editor);
        
        btnConfirmar = new JButton("Confirmar");
        btnBorrar = new JButton("Borrar");

        // Layout con GridBagConstraints para mejor alineación
        GridBagConstraints gbc = new GridBagConstraints();
        gbc.insets = new Insets(5, 5, 5, 5);
        gbc.fill = GridBagConstraints.HORIZONTAL;

        gbc.gridx = 0; gbc.gridy = 0;
        add(new JLabel("Medicamento:"), gbc);
        gbc.gridx = 1;
        add(txtMedicamento, gbc);

        gbc.gridx = 0; gbc.gridy = 1;
        add(new JLabel("Tipo:"), gbc);
        gbc.gridx = 1;
        add(cmbTipo, gbc);

        gbc.gridx = 0; gbc.gridy = 2;
        add(new JLabel("Cantidad:"), gbc);
        gbc.gridx = 1;
        add(txtCantidad, gbc);

        gbc.gridx = 0; gbc.gridy = 3;
        add(new JLabel("Distribuidor:"), gbc);
        gbc.gridx = 1;
        JPanel distribuidorPanel = new JPanel();
        distribuidorPanel.add(rbCofarma);
        distribuidorPanel.add(rbEmpsephar);
        distribuidorPanel.add(rbCemefar);
        add(distribuidorPanel, gbc);

        gbc.gridx = 0; gbc.gridy = 4;
        add(new JLabel("Sucursal:"), gbc);
        gbc.gridx = 1;
        JPanel sucursalPanel = new JPanel();
        sucursalPanel.add(chkPrincipal);
        sucursalPanel.add(chkSecundaria);
        add(sucursalPanel, gbc);

        gbc.gridx = 0; gbc.gridy = 5;
        add(new JLabel("Fecha del Pedido:"), gbc);
        gbc.gridx = 1;
        add(spinnerFecha, gbc);

        gbc.gridx = 0; gbc.gridy = 6;
        gbc.gridwidth = 2;
        JPanel buttonPanel = new JPanel();
        buttonPanel.add(btnConfirmar);
        buttonPanel.add(btnBorrar);
        add(buttonPanel, gbc);

        // Acción de botones
        btnConfirmar.addActionListener(e -> procesarPedido());
        btnBorrar.addActionListener(e -> limpiarFormulario());

        setVisible(true);
    }

    private void procesarPedido() {
        String medicamento = txtMedicamento.getText().trim();
        String tipo = (String) cmbTipo.getSelectedItem();
        String cantidadStr = txtCantidad.getText().trim();
        String distribuidor = rbCofarma.isSelected() ? "Cofarma" : rbEmpsephar.isSelected() ? "Empsephar" : rbCemefar.isSelected() ? "Cemefar" : "";
        String sucursal = (chkPrincipal.isSelected() ? "Principal" : "") + (chkSecundaria.isSelected() ? (chkPrincipal.isSelected() ? " y Secundaria" : "Secundaria") : "");
        String fechaPedido = ((JSpinner.DateEditor) spinnerFecha.getEditor()).getFormat().format(spinnerFecha.getValue());
        
        if (medicamento.isEmpty()) {
            JOptionPane.showMessageDialog(this, "Debe ingresar un nombre de medicamento.", "Error", JOptionPane.ERROR_MESSAGE);
            return;
        }
        
        if (distribuidor.isEmpty()) {
            JOptionPane.showMessageDialog(this, "Debe seleccionar un distribuidor.", "Error", JOptionPane.ERROR_MESSAGE);
            return;
        }
        
        if (sucursal.isEmpty()) {
            JOptionPane.showMessageDialog(this, "Debe seleccionar al menos una sucursal.", "Error", JOptionPane.ERROR_MESSAGE);
            return;
        }

        int cantidad;
        try {
            cantidad = Integer.parseInt(cantidadStr);
            if (cantidad <= 0) {
                throw new NumberFormatException();
            }
        } catch (NumberFormatException e) {
            JOptionPane.showMessageDialog(this, "Cantidad inválida. Debe ser un número entero positivo.", "Error", JOptionPane.ERROR_MESSAGE);
            return;
        }
        
        Pedido pedido = new Pedido(medicamento, tipo, cantidad, distribuidor, sucursal, fechaPedido);
        new ResumenPedidoGUI(pedido);
    }
    
    private void limpiarFormulario() {
    txtMedicamento.setText("");
    txtCantidad.setText("");
    cmbTipo.setSelectedIndex(0);
    rbCofarma.setSelected(false);
    rbEmpsephar.setSelected(false);
    rbCemefar.setSelected(false);
    chkPrincipal.setSelected(false);
    chkSecundaria.setSelected(false);
    spinnerFecha.setValue(new Date()); // Restablecer la fecha al día actual
}

}

// Clase que representa un Pedido

class Pedido {
    String medicamento, tipo, distribuidor, sucursal, fechaPedido;
    int cantidad;

    public Pedido(String medicamento, String tipo, int cantidad, String distribuidor, String sucursal, String fechaPedido) {
        this.medicamento = medicamento;
        this.tipo = tipo;
        this.cantidad = cantidad;
        this.distribuidor = distribuidor;
        this.sucursal = sucursal;
        this.fechaPedido = fechaPedido;
    }
}

// Clase para la ventana de resumen del pedido

class ResumenPedidoGUI extends JFrame {
    public ResumenPedidoGUI(Pedido pedido) {
        setTitle("Resumen del Pedido");
        setSize(400, 250);
        setLayout(new GridLayout(6, 1));
        
        add(new JLabel("Pedido al distribuidor: " + pedido.distribuidor));
        add(new JLabel(pedido.cantidad + " unidades del " + pedido.tipo + " " + pedido.medicamento));
        add(new JLabel("Para la sucursal: " + pedido.sucursal));
        add(new JLabel("Fecha del Pedido: " + pedido.fechaPedido));
        
        JButton btnCerrar = new JButton("Cerrar");
        btnCerrar.addActionListener(e -> dispose());
        add(btnCerrar);
        
        setVisible(true);
    }
}
