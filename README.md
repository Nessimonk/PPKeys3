using System;
using System.Collections.Generic;
using System.IO;
using System.Windows.Forms;
using System.Drawing;

namespace PP
{
    public partial class Form1 : Form
    {
        private double? firstOperand = null;
        private string operation = "";
        private bool isResultDisplayed = false;
        private List<string> history = new List<string>();
        private string savedResult = "";

        public Form1()
        {
            InitializeComponent();
            themeComboBox.SelectedIndexChanged += themeComboBox_SelectedIndexChanged;
            fontSizeTrackBar.Scroll += fontSizeTrackBar_Scroll;

            // Привязка событий к кнопкам (если не сделано в дизайнере)
            ShowHistory.Click += ShowHistory_Click;
            SaveResult.Click += SaveResult_Click;
            LoadResult.Click += LoadResult_Click;
        }

        private void AddDigit(string digit)
        {
            if (isResultDisplayed)
            {
                textBox1.Text = digit;
                isResultDisplayed = false;
                firstOperand = null;
            }
            else
            {
                textBox1.Text += digit;
            }
        }

        private void button1_Click(object sender, EventArgs e) => AddDigit("1");
        private void button2_Click(object sender, EventArgs e) => AddDigit("2");
        private void button3_Click(object sender, EventArgs e) => AddDigit("3");
        private void button4_Click(object sender, EventArgs e) => AddDigit("4");
        private void button5_Click(object sender, EventArgs e) => AddDigit("5");
        private void button6_Click(object sender, EventArgs e) => AddDigit("6");
        private void button7_Click(object sender, EventArgs e) => AddDigit("7");
        private void button8_Click(object sender, EventArgs e) => AddDigit("8");
        private void button9_Click(object sender, EventArgs e) => AddDigit("9");
        private void button10_Click(object sender, EventArgs e) => AddDigit("0");

        private void SetOperation(string opSymbol, string opName)
        {
            if (!double.TryParse(textBox1.Text, out double value))
            {
                MessageBox.Show("Введите число перед операцией.");
                return;
            }

            if (firstOperand == null || isResultDisplayed)
                firstOperand = value;

            operation = opName;
            textBox1.Text = firstOperand.ToString() + opSymbol;
            isResultDisplayed = false;
        }

        private void button14_Click(object sender, EventArgs e) => SetOperation("+", "add");
        private void button13_Click(object sender, EventArgs e) => SetOperation("-", "subtract");
        private void button15_Click(object sender, EventArgs e) => SetOperation("*", "multiply");
        private void button16_Click(object sender, EventArgs e) => SetOperation("/", "divide");

        private void button12_Click(object sender, EventArgs e)
        {
            string text = textBox1.Text;
            int opIndex = text.LastIndexOfAny(new char[] { '+', '-', '*', '/' });
            if (opIndex < 1 || opIndex >= text.Length - 1)
            {
                MessageBox.Show("Введите пример полностью.");
                return;
            }

            string opSymbol = text[opIndex].ToString();
            string left = text.Substring(0, opIndex);
            string right = text.Substring(opIndex + 1);

            if (!double.TryParse(left, out double leftOperand) || !double.TryParse(right, out double rightOperand))
            {
                MessageBox.Show("Некорректный пример.");
                return;
            }

            double result = 0;
            switch (opSymbol)
            {
                case "+": result = leftOperand + rightOperand; break;
                case "-": result = leftOperand - rightOperand; break;
                case "*": result = leftOperand * rightOperand; break;
                case "/":
                    if (rightOperand == 0)
                    {
                        MessageBox.Show("Деление на ноль запрещено.");
                        return;
                    }
                    result = leftOperand / rightOperand; break;
            }

            string example = $"{leftOperand}{opSymbol}{rightOperand}={result}";
            textBox1.Text = result.ToString();
            isResultDisplayed = true;
            firstOperand = result;

            // Добавляем в историю (но не показываем!)
            history.Add(example);
            textBoxHistory.Clear();
        }

        private void button11_Click(object sender, EventArgs e)
        {
            textBox1.Clear();
            firstOperand = null;
            operation = "";
            isResultDisplayed = false;
        }

        // 1. История только по кнопке, формат через ";  "
        private void ShowHistory_Click(object sender, EventArgs e)
        {
            if (history.Count == 0)
                textBoxHistory.Text = "История пуста.";
            else
                textBoxHistory.Text = string.Join(";  ", history);
        }

        // 2. Сохраняем только число (результат)
        private void SaveResult_Click(object sender, EventArgs e)
        {
            if (double.TryParse(textBox1.Text, out double result))
            {
                savedResult = result.ToString();
                File.WriteAllText("saved_result.txt", savedResult);
                MessageBox.Show("Результат сохранён.");
            }
            else
            {
                MessageBox.Show("Нет результата для сохранения.");
            }
        }

        // 3. Вставляем число, сохранённое SaveResult
        private void LoadResult_Click(object sender, EventArgs e)
        {
            if (File.Exists("saved_result.txt"))
            {
                savedResult = File.ReadAllText("saved_result.txt").Trim();
                // Если в textBox1 сейчас пример с операцией (например, "5+"), подставляем результат как второй операнд
                string text = textBox1.Text;
                int opIndex = text.LastIndexOfAny(new char[] { '+', '-', '*', '/' });
                if (opIndex >= 0 && opIndex == text.Length - 1)
                {
                    textBox1.Text += savedResult;
                }
                else
                {
                    textBox1.Text = savedResult;
                    firstOperand = double.TryParse(savedResult, out double val) ? val : (double?)null;
                    isResultDisplayed = true;
                }
                MessageBox.Show("Результат загружен.");
            }
            else
            {
                MessageBox.Show("Сохранённый результат не найден.");
            }
        }

        private void themeComboBox_SelectedIndexChanged(object sender, EventArgs e)
        {
            ComboBox cb = (ComboBox)sender;
            string selectedTheme = cb.SelectedItem.ToString();

            if (selectedTheme == "Светлая")
            {
                this.BackColor = SystemColors.Control;
                textBox1.BackColor = SystemColors.Window;
                textBox1.ForeColor = SystemColors.WindowText;
                textBoxHistory.BackColor = SystemColors.Window;
                textBoxHistory.ForeColor = SystemColors.WindowText;
            }
            else if (selectedTheme == "Темная")
            {
                this.BackColor = Color.Black;
                textBox1.BackColor = Color.FromArgb(30, 30, 30);
                textBox1.ForeColor = Color.Lime;
                textBoxHistory.BackColor = Color.FromArgb(30, 30, 30);
                textBoxHistory.ForeColor = Color.Lime;
            }
        }

        private void fontSizeTrackBar_Scroll(object sender, EventArgs e)
        {
            TrackBar tb = (TrackBar)sender;
            textBox1.Font = new Font(textBox1.Font.FontFamily, tb.Value, FontStyle.Regular);
            textBoxHistory.Font = new Font(textBoxHistory.Font.FontFamily, tb.Value, FontStyle.Regular);
        }
    }
}
