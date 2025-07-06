# hesap-makinesi
....
<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Modern Hesap Makinesi</title>
    <style>
        /* Google'dan modern bir font ekleyelim */
        @import url('https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;500&display=swap');

        :root {
            --bg-color: #121212;
            --primary-text-color: #FFFFFF;
            --display-bg-color: #1c1c1c; /* Arka planla uyumlu */
            --btn-dark-bg: #333333;
            --btn-orange-bg: #FF9500;
            --btn-light-bg: #A5A5A5;
            --btn-light-text: #000000;
        }

        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Poppins', sans-serif;
            background-color: var(--bg-color);
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
        }

        .calculator {
            background-color: var(--display-bg-color);
            width: 100%;
            max-width: 375px; /* Tipik bir telefon genişliği */
            padding: 25px;
            border-radius: 30px;
            box-shadow: 0 20px 40px rgba(0, 0, 0, 0.2);
            display: flex;
            flex-direction: column;
        }

        .display {
            width: 100%;
            margin-bottom: 20px;
            padding: 20px 10px;
            text-align: right;
            overflow-x: auto; /* Uzun sayılar için kaydırma */
            scrollbar-width: none; /* Firefox için scrollbar gizleme */
        }
        .display::-webkit-scrollbar {
            display: none; /* Chrome, Safari için scrollbar gizleme */
        }


        .display .current-operand {
            color: var(--primary-text-color);
            font-size: 64px;
            font-weight: 400;
            min-height: 80px;
            word-wrap: break-word;
            word-break: break-all;
        }

        .buttons {
            display: grid;
            grid-template-columns: repeat(4, 1fr);
            gap: 15px;
        }

        .btn {
            border: none;
            background-color: var(--btn-dark-bg);
            color: var(--primary-text-color);
            font-size: 28px;
            font-weight: 400;
            border-radius: 50%; /* Tam yuvarlak tuşlar */
            width: 70px;
            height: 70px;
            display: flex;
            justify-content: center;
            align-items: center;
            cursor: pointer;
            transition: background-color 0.2s;
            -webkit-tap-highlight-color: transparent; /* Mobil dokunma efektini kaldırma */
        }
        /* Geniş tuş için stil */
        .btn.span-two {
            grid-column: span 2;
            border-radius: 35px; /* Geniş tuş için daha oval bir görünüm */
            width: auto;
        }


        .btn:hover {
            filter: brightness(1.2);
        }

        .btn:active {
            filter: brightness(0.8);
        }
        
        /* Operatör tuşları */
        .btn.operator {
            background-color: var(--btn-orange-bg);
        }

        /* Üst sıradaki özel tuşlar */
        .btn.special {
            background-color: var(--btn-light-bg);
            color: var(--btn-light-text);
        }
        
    </style>
</head>
<body>

    <div class="calculator">
        <div class="display">
            <div class="current-operand" data-current-operand>0</div>
        </div>

        <div class="buttons">
            <button class="btn special" data-all-clear>AC</button>
            <button class="btn special" data-delete>⌫</button>
            <button class="btn special" data-operator>%</button>
            <button class="btn operator" data-operator>÷</button>

            <button class="btn" data-number>7</button>
            <button class="btn" data-number>8</button>
            <button class="btn" data-number>9</button>
            <button class="btn operator" data-operator>×</button>

            <button class="btn" data-number>4</button>
            <button class="btn" data-number>5</button>
            <button class="btn" data-number>6</button>
            <button class="btn operator" data-operator>-</button>
            
            <button class="btn" data-number>1</button>
            <button class="btn" data-number>2</button>
            <button class="btn" data-number>3</button>
            <button class="btn operator" data-operator>+</button>

            <button class="btn span-two" data-number>0</button>
            <button class="btn" data-number>,</button>
            <button class="btn operator" data-equals>=</button>
        </div>
    </div>

    <script>
        // JavaScript ile Hesap Makinesi Mantığı
        class Calculator {
            constructor(currentOperandTextElement) {
                this.currentOperandTextElement = currentOperandTextElement;
                this.clear();
            }

            // Her şeyi temizler
            clear() {
                this.currentOperand = '0';
                this.previousOperand = '';
                this.operation = undefined;
                this.updateDisplay();
            }

            // Son karakteri siler
            delete() {
                if (this.currentOperand.length > 1) {
                    this.currentOperand = this.currentOperand.toString().slice(0, -1);
                } else {
                    this.currentOperand = '0';
                }
                this.updateDisplay();
            }

            // Ekrana sayı veya virgül ekler
            appendNumber(number) {
                // Ekranda 0 varken yeni sayı yazıldığında 0'ı sil
                if (this.currentOperand === '0' && number !== ',') {
                    this.currentOperand = '';
                }
                // Bir sayıda birden fazla virgül olmasını engelle
                if (number === ',' && this.currentOperand.includes(',')) return;
                
                this.currentOperand = this.currentOperand.toString() + number.toString();
                this.updateDisplay();
            }

            // İşlem operatörünü seçer (+, -, ×, ÷)
            chooseOperation(operation) {
                if (this.currentOperand === '') return;
                if (this.previousOperand !== '') {
                    this.compute();
                }
                this.operation = operation;
                this.previousOperand = this.currentOperand;
                this.currentOperand = '';
            }

            // Hesaplamayı yapar
            compute() {
                let computation;
                // JavaScript'in ondalık sayıları doğru işlemesi için parseFloat kullanıyoruz
                // ve virgülü noktaya çeviriyoruz.
                const prev = parseFloat(this.previousOperand.replace(',', '.'));
                const current = parseFloat(this.currentOperand.replace(',', '.'));

                if (isNaN(prev) || isNaN(current)) return;

                switch (this.operation) {
                    case '+':
                        computation = prev + current;
                        break;
                    case '-':
                        computation = prev - current;
                        break;
                    case '×':
                        computation = prev * current;
                        break;
                    case '÷':
                        computation = prev / current;
                        break;
                     case '%':
                        // Yüzdeyi doğrudan uygular: 100 + 10% = 110
                        computation = prev * (current / 100);
                        break;
                    default:
                        return;
                }
                // Sonucu tekrar virgüle çevirerek atıyoruz
                this.currentOperand = computation.toString().replace('.', ',');
                this.operation = undefined;
                this.previousOperand = '';
            }

            // Ekranı günceller
            updateDisplay() {
                this.currentOperandTextElement.innerText = this.currentOperand;
            }
        }

        // HTML elementlerini seçme
        const numberButtons = document.querySelectorAll('[data-number]');
        const operatorButtons = document.querySelectorAll('[data-operator]');
        const equalsButton = document.querySelector('[data-equals]');
        const deleteButton = document.querySelector('[data-delete]');
        const allClearButton = document.querySelector('[data-all-clear]');
        const currentOperandTextElement = document.querySelector('[data-current-operand]');

        // Hesap makinesi nesnesini oluşturma
        const calculator = new Calculator(currentOperandTextElement);

        // Olay dinleyicilerini ekleme
        numberButtons.forEach(button => {
            button.addEventListener('click', () => {
                calculator.appendNumber(button.innerText);
            });
        });

        operatorButtons.forEach(button => {
            button.addEventListener('click', () => {
                calculator.chooseOperation(button.innerText);
            });
        });

        equalsButton.addEventListener('click', () => {
            calculator.compute();
            calculator.updateDisplay();
        });

        allClearButton.addEventListener('click', () => {
            calculator.clear();
        });

        deleteButton.addEventListener('click', () => {
            calculator.delete();
        });

    </script>

</body>
</html>
