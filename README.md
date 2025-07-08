<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Analise de Roleta Europeia</title>
<style>
  body {
    font-family: Arial, sans-serif;
    padding: 10px;
    max-width: 480px;
    margin: auto;
    background: #f0f0f0;
  }
  textarea {
    width: 100%;
    height: 120px;
    font-size: 16px;
  }
  button {
    margin-top: 10px;
    padding: 10px;
    font-size: 16px;
    cursor: pointer;
  }
  .freq-table {
    margin-top: 20px;
    width: 100%;
    border-collapse: collapse;
  }
  .freq-table th, .freq-table td {
    border: 1px solid #ccc;
    padding: 6px 8px;
    text-align: center;
  }
  .hot {
    background-color: #ff9999;
  }
  .cold {
    background-color: #99ccff;
  }
</style>
</head>
<body>

<h2>Análise de Frequência - Roleta Europeia</h2>
<p>Cole aqui os números sorteados da roleta (separados por vírgula):</p>
<textarea id="inputNumbers" placeholder="Ex: 32,7,0,12,36,7,12,32,0,15,7,3"></textarea>
<br />
<button onclick="analyze()">Analisar Frequência</button>

<div id="result"></div>

<script>
  function analyze() {
    const input = document.getElementById('inputNumbers').value;
    if(!input.trim()) {
      alert('Por favor, insira os números sorteados.');
      return;
    }

    // Processa os números
    const nums = input.split(',').map(n => parseInt(n.trim())).filter(n => !isNaN(n) && n >= 0 && n <= 36);

    if(nums.length === 0) {
      alert('Nenhum número válido encontrado. Use números entre 0 e 36.');
      return;
    }

    // Contagem de frequência
    const freq = {};
    for(let i=0; i<=36; i++) freq[i] = 0;

    nums.forEach(n => freq[n]++);

    // Converte para array e ordena por frequência desc
    const freqArr = Object.entries(freq).sort((a,b) => b[1] - a[1]);

    // Define hot e cold (top 5 e bottom 5)
    const hotNumbers = freqArr.slice(0,5).map(x => parseInt(x[0]));
    const coldNumbers = freqArr.slice(-5).map(x => parseInt(x[0]));

    // Monta tabela HTML
    let html = '<table class="freq-table"><tr><th>Número</th><th>Frequência</th></tr>';
    freqArr.forEach(([num, count]) => {
      const n = parseInt(num);
      const cls = hotNumbers.includes(n) ? 'hot' : (coldNumbers.includes(n) ? 'cold' : '');
      html += `<tr class="${cls}"><td>${num}</td><td>${count}</td></tr>`;
    });
    html += '</table>';

    // Sugestão de números para apostar (os hot)
    html += '<h3>Números sugeridos para apostar (mais frequentes):</h3>';
    html += '<p>' + hotNumbers.join(', ') + '</p>';

    document.getElementById('result').innerHTML = html;
  }
</script>

</body>
</html>
