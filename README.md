<!DOCTYPE html>
<html>
<head>
<style>
  .container {
    display: flex;
    justify-content: center;
    align-items: center;
    font-family: "Microsoft YaHei", sans-serif;
    background-color: #f9f9f9;
    padding: 20px;
  }
  svg {
    max-width: 500px;
    height: auto;
    border: 1px solid #ddd;
    background: white;
  }
  .label {
    font-size: 14px;
    fill: #333;
  }
  .pointer {
    stroke: black;
    stroke-width: 1;
    fill: none;
  }
  .outline {
    stroke: black;
    stroke-width: 2;
    fill: none;
    stroke-linecap: round;
  }
</style>
</head>
<body>

<div class="container">
  <svg viewBox="0 0 400 450" xmlns="http://www.w3.org/2000/svg">
    <path class="outline" d="M50,100 Q80,20 150,20 T250,80 L250,250 Q250,400 200,430" /> <path class="outline" d="M50,100 L40,150 L60,160 Q40,250 100,280 Q150,300 180,430" /> <path class="outline" d="M70,180 Q150,180 230,220" /> <path class="outline" d="M80,260 Q150,220 220,260 Q200,310 120,300 Z" fill="#f0f0f0" /> <path class="outline" d="M220,260 Q240,280 230,320" /> <path class="outline" d="M250,250 L250,430" /> <line class="pointer" x1="100" y1="60" x2="120" y2="100" />
    <text x="80" y="55" class="label">鼻腔</text>
    
    <line class="pointer" x1="220" y1="100" x2="200" y2="175" />
    <text x="215" y="95" class="label">腭</text>
    
    <text x="145" y="260" class="label" style="font-weight:bold; font-size:18px;">舌</text>
    
    <line class="pointer" x1="270" y1="150" x2="210" y2="190" />
    <text x="275" y="150" class="label">口腔</text>
    
    <line class="pointer" x1="280" y1="285" x2="235" y2="285" />
    <text x="285" y="290" class="label">會厭</text>
    
    <line class="pointer" x1="180" y1="380" x2="210" y2="380" />
    <text x="160" y="385" class="label">喉</text>
    
    <line class="pointer" x1="310" y1="400" x2="260" y2="420" />
    <text x="315" y="405" class="label">食管</text>
  </svg>
</div>

</body>
</html>
