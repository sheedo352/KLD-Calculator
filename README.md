package Activity1;

import javax.swing.*;
import java.awt.*;
import java.awt.event.*;
import java.util.ArrayList;
import java.util.Arrays;

public class ICDIfy extends JFrame implements ActionListener {

    private final JTextField display;

    private String expression = "";

    private final ArrayList<String> historyList = new ArrayList<>();

    private final Color bgColor = new Color(255, 255, 255);
    private final Color buttonColor = new Color(0, 153, 76);
    private final Color accentColor = new Color(255, 204, 0);
    private final Color textColor = Color.WHITE;

    public ICDIfy() {

        setTitle("ICDIfy");
        setSize(320, 450);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setLocationRelativeTo(null);
        setLayout(new BorderLayout(5, 5));

        Image icon = Toolkit.getDefaultToolkit().getImage("C:\\Users\\estol\\Downloads\\icon.png");
        setIconImage(icon);

        JPanel northPanel = new JPanel(new BorderLayout());

        JPanel topBar = new JPanel(new BorderLayout());
        topBar.setBackground(Color.WHITE);
        topBar.setBorder(BorderFactory.createEmptyBorder(5, 5, 5, 5));

        ImageIcon historyIcon = new ImageIcon("C:\\Users\\estol\\Downloads\\history.png");
        Image img = historyIcon.getImage().getScaledInstance(20, 20, Image.SCALE_SMOOTH);
        historyIcon = new ImageIcon(img);

        JButton historyBtn = new JButton(historyIcon);
        historyBtn.setFocusPainted(false);
        historyBtn.setBorderPainted(false);
        historyBtn.setContentAreaFilled(false);
        historyBtn.setCursor(new Cursor(Cursor.HAND_CURSOR));
        historyBtn.addActionListener(e -> showHistory());

        topBar.add(historyBtn, BorderLayout.WEST);

        display = new JTextField("0");
        display.setFont(new Font("Segoe UI", Font.BOLD, 28));
        display.setHorizontalAlignment(JTextField.RIGHT);
        display.setEditable(false);
        display.setBackground(bgColor);
        display.setBorder(BorderFactory.createEmptyBorder(10, 10, 10, 10));

        northPanel.add(topBar, BorderLayout.NORTH);
        northPanel.add(display, BorderLayout.CENTER);

        add(northPanel, BorderLayout.NORTH);

        JPanel panel = new JPanel(new GridLayout(6, 4, 5, 5));
        panel.setBackground(bgColor);

        String[] buttons = {
                "%", "CE", "C", "DEL",
                "1/x", "x²", "√x", "÷",
                "7", "8", "9", "×",
                "4", "5", "6", "−",
                "1", "2", "3", "+",
                "+/-", "0", ".", "="
        };

        for (String text : buttons) {
            JButton btn = new JButton(text);
            btn.setFont(new Font("Segoe UI", Font.BOLD, 14));
            btn.setFocusPainted(false);
            btn.setBackground(buttonColor);
            btn.setForeground(textColor);

            if ("÷×−+".contains(text)) {
                btn.setBackground(accentColor);
                btn.setForeground(Color.BLACK);
            }

            if (text.equals("=")) {
                btn.setBackground(Color.YELLOW);
                btn.setForeground(Color.BLACK);
            }

            btn.addActionListener(this);
            panel.add(btn);
        }

        add(panel, BorderLayout.CENTER);
    }

    private String formatResult(double result) {
        if (result == (long) result) {
            return String.valueOf((long) result);
        } else {
            return String.valueOf(result);
        }
    }

    @Override
    public void actionPerformed(ActionEvent e) {

        String cmd = ((JButton) e.getSource()).getText();

        switch (cmd) {

            case "1/x" -> {
                try {
                    double value = Double.parseDouble(expression);
                    if (value != 0) {
                        String formatted = formatResult(1 / value);
                        display.setText(formatted);
                        expression = formatted;
                    } else {
                        display.setText("Error");
                        expression = "";
                    }
                } catch (Exception ex) {
                    display.setText("Error");
                    expression = "";
                }
            }

            case "x²" -> {
                try {
                    double value = Double.parseDouble(expression);
                    String formatted = formatResult(value * value);
                    display.setText(formatted);
                    expression = formatted;
                } catch (Exception ex) {
                    display.setText("Error");
                    expression = "";
                }
            }

            case "√x" -> {
                try {
                    double value = Double.parseDouble(expression);
                    if (value >= 0) {
                        String formatted = formatResult(Math.sqrt(value));
                        display.setText(formatted);
                        expression = formatted;
                    } else {
                        display.setText("Error");
                        expression = "";
                    }
                } catch (Exception ex) {
                    display.setText("Error");
                    expression = "";
                }
            }

            case "C", "CE" -> {
                expression = "";
                display.setText("0");
            }

            case "DEL" -> {
                if (!expression.isEmpty()) {
                    expression = expression.substring(0, expression.length() - 1);
                    display.setText(expression.isEmpty() ? "0" : expression);
                }
            }

            case "+", "−", "×", "÷", "%" -> {
                expression += " " + cmd + " ";
                display.setText(expression);
            }

            case "=" -> {
                try {
                    String exp = expression.replace("−", "-");
                    ArrayList<String> list = new ArrayList<>(Arrays.asList(exp.split(" ")));

                    for (int i = 0; i < list.size(); i++) {
                        String op = list.get(i);
                        if (op.equals("×") || op.equals("÷") || op.equals("%")) {
                            double a = Double.parseDouble(list.get(i - 1));
                            double b = Double.parseDouble(list.get(i + 1));
                            double res = 0;

                            switch (op) {
                                case "×" -> res = a * b;
                                case "÷" -> res = (b != 0) ? a / b : 0;
                                case "%" -> res = (a * b) / 100;
                            }

                            list.set(i - 1, String.valueOf(res));
                            list.remove(i);
                            list.remove(i);
                            i--;
                        }
                    }

                    double result = Double.parseDouble(list.get(0));
                    for (int i = 1; i < list.size(); i += 2) {
                        double num = Double.parseDouble(list.get(i + 1));
                        if (list.get(i).equals("+")) result += num;
                        else result -= num;
                    }

                    String formatted = formatResult(result);

                    historyList.add(0, expression + " = " + formatted);
                    if (historyList.size() > 10) historyList.remove(10);

                    display.setText(formatted);
                    expression = formatted;

                } catch (Exception ex) {
                    display.setText("Error");
                    expression = "";
                }
            }

            case "." -> {
                expression += ".";
                display.setText(expression);
            }

            case "+/-" -> {
                expression = "-(" + expression + ")";
                display.setText(expression);
            }

            default -> {

                String[] parts = expression.split("[+\\-×÷% ]");
                String last = parts.length > 0 ? parts[parts.length - 1] : "";

                if (last.replace(".", "").replace("-", "").length() >= 10) return;

                expression = (expression.equals("") || expression.equals("0")) ? cmd : expression + cmd;
                display.setText(expression);
            }
        }

        if (expression.equals("100 + 55")) {
            display.setText("IMISSYOU");
            expression = "";
        }
    }

    private void showHistory() {

        if (historyList.isEmpty()) {
            JOptionPane.showMessageDialog(this, "No history yet");
            return;
        }

        JTextArea area = new JTextArea();
        area.setEditable(false);
        area.setFont(new Font("Segoe UI", Font.PLAIN, 14));
        area.setText(String.join("\n", historyList));

        JScrollPane scroll = new JScrollPane(area);
        scroll.setPreferredSize(new Dimension(250, 300));

        JOptionPane.showMessageDialog(this, scroll, "History", JOptionPane.PLAIN_MESSAGE);
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> new ICDIfy().setVisible(true));
    }
}
