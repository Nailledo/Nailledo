$ErrorActionPreference = 'Stop'
[Threading.Thread]::CurrentThread.CurrentCulture = [Globalization.CultureInfo]::InvariantCulture
$root = Split-Path -Parent $MyInvocation.MyCommand.Path
$out = Join-Path $root 'assets'
New-Item -ItemType Directory -Force $out | Out-Null

function Save($name, $content) {
    [IO.File]::WriteAllText((Join-Path $out $name), $content, (New-Object Text.UTF8Encoding($false)))
}
function Len($s) { [Net.WebUtility]::HtmlDecode($s).Length }
function Pc($t, $c) { [math]::Round($t / $c * 100, 3) }

$MONO = "'JetBrains Mono','Fira Code',ui-monospace,SFMono-Regular,Consolas,'Liberation Mono',Menlo,monospace"

# ---------- shared building blocks ----------
function Stars($w, $h, $n, $seed) {
    $r = New-Object Random $seed
    $sb = New-Object Text.StringBuilder
    $cols = @('#ffffff', '#e9d5ff', '#a5f3fc', '#fbcfe8')
    for ($i = 0; $i -lt $n; $i++) {
        $x = [math]::Round($r.NextDouble() * $w, 1)
        $y = [math]::Round($r.NextDouble() * $h, 1)
        $rad = [math]::Round(0.5 + $r.NextDouble() * 1.4, 2)
        $d = [math]::Round($r.NextDouble() * 4, 2)
        $du = [math]::Round(2 + $r.NextDouble() * 3, 2)
        $c = $cols[$r.Next(4)]
        [void]$sb.Append("<circle class='st' cx='$x' cy='$y' r='$rad' fill='$c' style='animation-duration:${du}s;animation-delay:${d}s'/>")
    }
    $sb.ToString()
}

function Orbs($w, $yMin, $yMax, $n, $seed) {
    $r = New-Object Random $seed
    $sb = New-Object Text.StringBuilder
    $cols = @('#c084fc', '#f472b6', '#22d3ee', '#a855f7')
    for ($i = 0; $i -lt $n; $i++) {
        $x = [math]::Round($r.NextDouble() * $w, 1)
        $y = [math]::Round($yMin + $r.NextDouble() * ($yMax - $yMin), 1)
        $rad = [math]::Round(2 + $r.NextDouble() * 4, 1)
        $d = [math]::Round($r.NextDouble() * 10, 2)
        $du = [math]::Round(7 + $r.NextDouble() * 8, 2)
        $c = $cols[$r.Next(4)]
        [void]$sb.Append("<circle class='orb' cx='$x' cy='$y' r='$rad' fill='$c' style='animation-duration:${du}s;animation-delay:${d}s'/>")
    }
    $sb.ToString()
}

function Wave($y, $amp, $period, $fill, $op, $dur, $rev, $h) {
    $half = $period / 2
    $segs = [int](2000 / $half)
    $d = "M0,0 q$($half / 2),$(-2 * $amp) $half,0" + (" t$half,0" * ($segs - 1)) + " L2000,$h L0,$h Z"
    $dir = if ($rev) { 'reverse' } else { 'normal' }
    "<g transform='translate(0,$y)'><path d='$d' fill='$fill' fill-opacity='$op' style='animation:slide ${dur}s linear infinite;animation-direction:$dir'/></g>"
}

$COMMON_CSS = @"
.st{animation:tw 3s ease-in-out infinite}
@keyframes tw{0%,100%{opacity:.12}50%{opacity:1}}
@keyframes slide{to{transform:translateX(-1000px)}}
.orb{animation:rise 10s linear infinite;opacity:0}
@keyframes rise{0%{transform:translateY(0);opacity:0}15%{opacity:.75}100%{transform:translateY(-280px);opacity:0}}
@keyframes blink{0%,49%{opacity:1}50%,100%{opacity:0}}
"@

# =====================================================================
# HEADER
# =====================================================================
$W = 1000; $H = 340
$letters = 'JONATHAN LECLERC'
$cell = 44
$x0 = 500 - ($letters.Length * $cell) / 2 + $cell / 2
$letterSvg = ''
for ($i = 0; $i -lt $letters.Length; $i++) {
    $ch = $letters[$i]
    if ($ch -eq ' ') { continue }
    $x = $x0 + $i * $cell
    $dl = [math]::Round(0.35 + $i * 0.07, 2)
    $wl = [math]::Round(2.2 + $i * 0.14, 2)
    $letterSvg += "<g class='drop' style='animation-delay:${dl}s'><g class='wave' style='animation-delay:${wl}s'><text x='$x' y='158' text-anchor='middle' font-size='68' font-weight='800' fill='url(#ng)' stroke='#ffffff' stroke-opacity='.25' stroke-width='.8'>$ch</text></g></g>"
}

$tag = '&lt;hello world /&gt;'
$tagLen = (Len $tag) * 11
$sub = 'BUT Informatique &#183; Apprenti IES Ing&#233;nierie 2026/27 &#183; Le Havre'
$subLen = (Len $sub) * 9.6

$header = @"
<svg xmlns='http://www.w3.org/2000/svg' width='$W' height='$H' viewBox='0 0 $W $H' role='img' aria-label='Jonathan LECLERC alias Nailt'>
<defs>
<linearGradient id='bg' x1='0' y1='0' x2='1' y2='1'><stop offset='0' stop-color='#07010f'/><stop offset='.55' stop-color='#1a0636'/><stop offset='1' stop-color='#2e0b5c'/></linearGradient>
<linearGradient id='ng' gradientUnits='userSpaceOnUse' x1='150' y1='0' x2='850' y2='0' spreadMethod='repeat'>
<stop offset='0' stop-color='#c084fc'/><stop offset='.25' stop-color='#f472b6'/><stop offset='.5' stop-color='#22d3ee'/><stop offset='.75' stop-color='#f472b6'/><stop offset='1' stop-color='#c084fc'/>
<animateTransform attributeName='gradientTransform' type='translate' from='0 0' to='700 0' dur='5s' repeatCount='indefinite'/>
</linearGradient>
<linearGradient id='ag' gradientUnits='userSpaceOnUse' x1='0' y1='0' x2='1000' y2='0' spreadMethod='repeat'>
<stop offset='0' stop-color='#22d3ee'/><stop offset='.5' stop-color='#a855f7'/><stop offset='1' stop-color='#22d3ee'/>
<animateTransform attributeName='gradientTransform' type='translate' from='0 0' to='1000 0' dur='6s' repeatCount='indefinite'/>
</linearGradient>
<filter id='glow' x='-20%' y='-60%' width='140%' height='220%'><feGaussianBlur stdDeviation='7' result='b'/><feMerge><feMergeNode in='b'/><feMergeNode in='b'/><feMergeNode in='SourceGraphic'/></feMerge></filter>
<filter id='blur' x='-50%' y='-50%' width='200%' height='200%'><feGaussianBlur stdDeviation='45'/></filter>
<pattern id='scan' width='4' height='4' patternUnits='userSpaceOnUse'><rect width='4' height='1' fill='#ffffff' fill-opacity='.045'/></pattern>
<clipPath id='card'><rect width='$W' height='$H' rx='26'/></clipPath>
</defs>
<style>
text{font-family:$MONO}
$COMMON_CSS
.a1{animation:d1 12s ease-in-out infinite}.a2{animation:d2 15s ease-in-out infinite}.a3{animation:d3 10s ease-in-out infinite}
@keyframes d1{0%,100%{transform:translate(0,0)}50%{transform:translate(260px,40px)}}
@keyframes d2{0%,100%{transform:translate(0,0)}50%{transform:translate(-300px,-30px)}}
@keyframes d3{0%,100%{transform:translate(0,0)}50%{transform:translate(120px,-50px)}}
.drop{animation:drop .9s cubic-bezier(.2,1.5,.4,1) both}
@keyframes drop{from{transform:translateY(-80px);opacity:0}to{transform:translateY(0);opacity:1}}
.wave{animation:wave 3.2s ease-in-out infinite}
@keyframes wave{0%,100%{transform:translateY(0)}50%{transform:translateY(-10px)}}
.fl{animation:fl 7s ease-in-out infinite}
@keyframes fl{0%,100%{transform:translateY(0)}50%{transform:translateY(-14px)}}
.fadein{animation:fi 1.2s ease 1.6s both}
@keyframes fi{from{opacity:0;transform:translateY(8px)}to{opacity:1;transform:translateY(0)}}
.g1,.g2{opacity:0}
.g1{animation:gl1 4.2s steps(1) infinite}.g2{animation:gl2 4.2s steps(1) infinite}.gm{animation:glm 4.2s steps(1) infinite}
@keyframes gl1{0%,86%{opacity:0;transform:translate(0,0)}87%{opacity:.9;transform:translate(-5px,1px)}89%{opacity:.9;transform:translate(4px,-1px)}91%{opacity:.9;transform:translate(-3px,2px)}93%{opacity:0}100%{opacity:0}}
@keyframes gl2{0%,86%{opacity:0;transform:translate(0,0)}88%{opacity:.9;transform:translate(5px,-1px)}90%{opacity:.9;transform:translate(-4px,1px)}92%{opacity:.9;transform:translate(3px,0)}94%{opacity:0}100%{opacity:0}}
@keyframes glm{0%,86%{transform:skewX(0)}88%{transform:skewX(-8deg)}90%{transform:skewX(6deg)}92%{transform:skewX(-3deg)}94%,100%{transform:skewX(0)}}
.bar{transform-origin:center;transform-box:fill-box;animation:bar 4s ease-in-out infinite}
@keyframes bar{0%,100%{transform:scaleX(.35);opacity:.5}50%{transform:scaleX(1);opacity:1}}
</style>
<g clip-path='url(#card)'>
<rect width='$W' height='$H' fill='url(#bg)'/>
<g filter='url(#blur)'>
<ellipse class='a1' cx='180' cy='120' rx='190' ry='90' fill='#7c3aed' fill-opacity='.55'/>
<ellipse class='a2' cx='780' cy='90' rx='200' ry='80' fill='#db2777' fill-opacity='.35'/>
<ellipse class='a3' cx='480' cy='250' rx='220' ry='70' fill='#0891b2' fill-opacity='.35'/>
</g>
$(Stars $W $H 70 7)
<text x='36' y='290' font-size='230' font-weight='800' fill='#a855f7' fill-opacity='.07'>{</text>
<text x='830' y='290' font-size='230' font-weight='800' fill='#a855f7' fill-opacity='.07'>}</text>
$(Orbs $W 250 340 16 11)
<g class='fadein'><text x='$((1000 - $tagLen) / 2)' y='66' font-size='18' fill='#22d3ee' textLength='$tagLen' lengthAdjust='spacing'>$tag</text></g>
<g filter='url(#glow)'>$letterSvg</g>
<g class='fl'>
<g class='fadein' style='animation-delay:2.2s'>
<rect class='bar' x='250' y='200' width='150' height='3' rx='1.5' fill='url(#ag)'/>
<rect class='bar' x='600' y='200' width='150' height='3' rx='1.5' fill='url(#ag)' style='animation-delay:-2s'/>
<g class='gm' style='transform-origin:500px 218px'>
<text class='g1' x='500' y='222' text-anchor='middle' font-size='46' font-weight='800' fill='#22d3ee' textLength='190' lengthAdjust='spacing'>NAILT</text>
<text class='g2' x='500' y='222' text-anchor='middle' font-size='46' font-weight='800' fill='#f472b6' textLength='190' lengthAdjust='spacing'>NAILT</text>
<text x='500' y='222' text-anchor='middle' font-size='46' font-weight='800' fill='#ffffff' textLength='190' lengthAdjust='spacing' style='filter:drop-shadow(0 0 8px #a855f7)'>NAILT</text>
</g>
</g>
</g>
<g class='fadein' style='animation-delay:2.8s'><text x='$((1000 - $subLen) / 2)' y='264' font-size='16' fill='#e9d5ff' fill-opacity='.9' textLength='$subLen' lengthAdjust='spacing'>$sub</text></g>
$(Wave 292 14 500 '#6A0DAD' 0.55 14 $false 60)
$(Wave 300 10 250 '#a855f7' 0.35 9 $true 50)
$(Wave 310 8 200 '#22d3ee' 0.18 7 $false 40)
<rect width='$W' height='$H' fill='url(#scan)'/>
</g>
<rect x='1' y='1' width='998' height='338' rx='25' fill='none' stroke='url(#ag)' stroke-width='2' stroke-opacity='.7'/>
</svg>
"@
Save 'header.svg' $header

# =====================================================================
# TYPING (rotating typed lines)
# =====================================================================
function TypedCss($id, $Wpx, $n, $t0, $t1, $t2, $t3, $curEnd, $C) {
    $p0 = Pc $t0 $C; $p1 = Pc $t1 $C; $p2 = Pc $t2 $C; $p3 = Pc $t3 $C; $pe = Pc $curEnd $C
    @"
@keyframes ty$id{0%{transform:translateX(0)}$p0%{transform:translateX(0)}$p1%{transform:translateX(${Wpx}px)}$p2%{transform:translateX(${Wpx}px)}$p3%{transform:translateX(0)}100%{transform:translateX(0)}}
.ty$id{animation:ty$id ${C}s steps($n) infinite}
@keyframes vi$id{0%{opacity:0}$p0%{opacity:1}$pe%{opacity:0}100%{opacity:0}}
.vi$id{animation:vi$id ${C}s step-end infinite}
"@
}

$tl = @(
    '&#201;tudiant en BUT Informatique &#183; 3e ann&#233;e',
    'Futur apprenti chez IES Ing&#233;nierie &#183; 2026/2027',
    'D&#233;veloppeur web &amp; applicatif &#183; Java &#183; C# &#183; PHP',
    'Fan de Linux, Docker, Unity &amp; Fusion 360',
    'Toujours pr&#234;t &#224; apprendre de nouvelles choses'
)
$per = 5.0; $C = $tl.Count * $per; $cw = 14.4
$css = ''; $body = ''
for ($i = 0; $i -lt $tl.Count; $i++) {
    $n = Len $tl[$i]; $Wpx = [math]::Round($n * $cw, 2)
    $xs = [math]::Round((1000 - $Wpx) / 2, 2)
    $s = $i * $per
    $css += TypedCss $i $Wpx $n $s ($s + 2.0) ($s + 4.0) ($s + 4.7) ($s + 4.7) $C
    $body += @"
<clipPath id='c$i'><rect class='ty$i' x='$($xs - $Wpx)' y='0' width='$Wpx' height='70'/></clipPath>
<text x='$xs' y='44' font-size='24' font-weight='600' fill='url(#tg)' textLength='$Wpx' lengthAdjust='spacing' clip-path='url(#c$i)'>$($tl[$i])</text>
<g class='ty$i'><g class='vi$i'><rect x='$($xs + 2)' y='16' width='3' height='34' rx='1.5' fill='#f472b6' style='animation:blink 1s step-end infinite'/></g></g>
"@
}
$typing = @"
<svg xmlns='http://www.w3.org/2000/svg' width='1000' height='70' viewBox='0 0 1000 70' role='img' aria-label='Presentation'>
<defs><linearGradient id='tg' gradientUnits='userSpaceOnUse' x1='100' y1='0' x2='900' y2='0'><stop offset='0' stop-color='#22d3ee'/><stop offset='.5' stop-color='#c084fc'/><stop offset='1' stop-color='#f472b6'/></linearGradient></defs>
<style>
text{font-family:$MONO}
@keyframes blink{0%,49%{opacity:1}50%,100%{opacity:0}}
$css
</style>
$body
</svg>
"@
Save 'typing.svg' $typing

# =====================================================================
# TERMINAL
# =====================================================================
function Spn($c, $t) { [pscustomobject]@{ c = $c; t = $t } }
function SpLen($spans) { $n = 0; foreach ($s in $spans) { $n += (Len $s.t) }; $n }
function SpText($x, $y, $spans, $cw, $extra) {
    $n = SpLen $spans; $inner = ''
    foreach ($s in $spans) { $inner += "<tspan fill='$($s.c)'>$($s.t)</tspan>" }
    "<text x='$x' y='$y' font-size='18' textLength='$([math]::Round($n * $cw, 2))' lengthAdjust='spacing' $extra>$inner</text>"
}
function FadeCss($id, $ta, $C, $endShow, $endHide) {
    $pa = Pc $ta $C; $pb = Pc ($ta + 0.3) $C; $pe = Pc $endShow $C; $pf = Pc $endHide $C
    "@keyframes fd$id{0%{opacity:0}$pa%{opacity:0}$pb%{opacity:1}$pe%{opacity:1}$pf%{opacity:0}100%{opacity:0}}.fd$id{animation:fd$id ${C}s ease infinite;opacity:0}"
}

$cwT = 10.8; $tx = 44; $TC = 17.0
$G = '#4ade80'; $B = '#60a5fa'; $Wt = '#f3e8ff'; $Gy = '#94a3b8'; $P = '#c084fc'; $S = '#86efac'
$prompt = @((Spn $G 'nailt@lehavre'), (Spn $Wt ':'), (Spn $B '~'), (Spn $Wt '$ '))
$cmd1 = $prompt + @((Spn $Wt 'whoami'))
$cmd2 = $prompt + @((Spn $Wt 'cat profil.json'))
$out1 = @((Spn $P 'Jonathan LECLERC'), (Spn $Gy ' &#8212; alias '), (Spn '#f472b6' 'Nailt'))
$json = @(
    @{ ind = 0; sp = @((Spn $Gy '{')) },
    @{ ind = 2; sp = @((Spn $P '"formation"'), (Spn $Gy ': '), (Spn $S '"BUT Informatique &#183; 3e ann&#233;e"'), (Spn $Gy ',')) },
    @{ ind = 2; sp = @((Spn $P '"alternance"'), (Spn $Gy ': '), (Spn $S '"IES Ing&#233;nierie &#183; 2026/2027"'), (Spn $Gy ',')) },
    @{ ind = 2; sp = @((Spn $P '"lieu"'), (Spn $Gy ': '), (Spn $S '"Port-J&#233;r&#244;me-sur-Seine"'), (Spn $Gy ',')) },
    @{ ind = 2; sp = @((Spn $P '"ville"'), (Spn $Gy ': '), (Spn $S '"Le Havre, France"')) },
    @{ ind = 0; sp = @((Spn $Gy '}')) }
)

$css = ''; $body = ''
# cmd1
$n = SpLen $cmd1; $Wpx = [math]::Round($n * $cwT, 2)
$css += TypedCss 'a' $Wpx $n 0.6 1.6 14.0 14.01 1.9 $TC
$body += "<clipPath id='ca'><rect class='tya' x='$($tx - $Wpx)' y='60' width='$Wpx' height='40'/></clipPath>" + (SpText $tx 90 $cmd1 $cwT "clip-path='url(#ca)'") + "<g class='tya'><g class='via'><rect x='$($tx + 1)' y='73' width='9' height='20' fill='#f3e8ff' style='animation:blink 1s step-end infinite'/></g></g>"
# out1
$css += FadeCss 'o1' 1.9 $TC 14.0 14.6
$body += "<g class='fdo1'>" + (SpText $tx 120 $out1 $cwT '') + '</g>'
# cmd2
$n = SpLen $cmd2; $Wpx = [math]::Round($n * $cwT, 2)
$css += TypedCss 'b' $Wpx $n 2.8 4.0 14.0 14.01 4.3 $TC
$body += "<clipPath id='cb'><rect class='tyb' x='$($tx - $Wpx)' y='140' width='$Wpx' height='40'/></clipPath>" + (SpText $tx 170 $cmd2 $cwT "clip-path='url(#cb)'") + "<g class='tyb'><g class='vib'><rect x='$($tx + 1)' y='153' width='9' height='20' fill='#f3e8ff' style='animation:blink 1s step-end infinite'/></g></g>"
# json
$jy = 200
for ($i = 0; $i -lt $json.Count; $i++) {
    $j = $json[$i]
    $css += FadeCss "j$i" (4.6 + $i * 0.35) $TC 14.0 14.6
    $body += "<g class='fdj$i'>" + (SpText ($tx + $j.ind * $cwT) ($jy + $i * 26) $j.sp $cwT '') + '</g>'
}
# final prompt
$css += FadeCss 'p' 7.0 $TC 14.0 14.6
$py = $jy + $json.Count * 26 + 8
$pn = SpLen $prompt
$body += "<g class='fdp'>" + (SpText $tx $py $prompt $cwT '') + "<rect x='$($tx + $pn * $cwT + 2)' y='$($py - 16)' width='10' height='21' fill='#f472b6' style='animation:blink 1s step-end infinite'/></g>"

$term = @"
<svg xmlns='http://www.w3.org/2000/svg' width='1000' height='$($py + 36)' viewBox='0 0 1000 $($py + 36)' role='img' aria-label='Terminal profil'>
<defs>
<linearGradient id='tb' gradientUnits='userSpaceOnUse' x1='0' y1='0' x2='1000' y2='0'><stop offset='0' stop-color='#a855f7'/><stop offset='.5' stop-color='#22d3ee'/><stop offset='1' stop-color='#f472b6'/>
<animateTransform attributeName='gradientTransform' type='rotate' from='0 500 200' to='360 500 200' dur='8s' repeatCount='indefinite'/></linearGradient>
<filter id='gl' x='-5%' y='-5%' width='110%' height='110%'><feGaussianBlur stdDeviation='8'/></filter>
</defs>
<style>
text{font-family:$MONO}
@keyframes blink{0%,49%{opacity:1}50%,100%{opacity:0}}
$css
</style>
<rect x='10' y='10' width='980' height='$($py + 16)' rx='16' fill='none' stroke='url(#tb)' stroke-width='6' opacity='.45' filter='url(#gl)'/>
<rect x='10' y='10' width='980' height='$($py + 16)' rx='16' fill='#0b0614' stroke='url(#tb)' stroke-width='2'/>
<path d='M10,52 V26 a16,16 0 0 1 16,-16 H974 a16,16 0 0 1 16,16 V52 Z' fill='#170b2b'/>
<circle cx='40' cy='31' r='7' fill='#ff5f57'/><circle cx='64' cy='31' r='7' fill='#febc2e'/><circle cx='88' cy='31' r='7' fill='#28c840'/>
<text x='500' y='36' text-anchor='middle' font-size='14' fill='#8b7aa8' textLength='230' lengthAdjust='spacing'>nailt@lehavre: ~/profil</text>
$body
</svg>
"@
Save 'terminal.svg' $term

# =====================================================================
# SKILLS MARQUEE
# =====================================================================
$rows = @(
    @(@('HTML5', '#e34f26'), @('CSS3', '#1572b6'), @('JavaScript', '#f7df1e'), @('Bootstrap', '#9a6bf0'), @('PHP', '#8892bf'), @('Laravel', '#ff2d20'), @('Tailwind', '#38bdf8'), @('Twig', '#bacf29')),
    @(@('Java', '#f89820'), @('Android Studio', '#3ddc84'), @('C#', '#a179dc'), @('Python', '#ffd43b'), @('C', '#a8b9cc'), @('MySQL', '#4aa3d8'), @('PostgreSQL', '#5b8fc9'), @('PL/pgSQL', '#8aa7d6')),
    @(@('Linux', '#fcc624'), @('Bash', '#4eaa25'), @('Git', '#f05032'), @('Docker', '#2496ed'), @('VS Code', '#2b9be0'), @('Laragon', '#0e9ce0'), @('Fusion 360', '#ff5a36'), @('Unity', '#e5e7eb'), @('Scratch', '#ff8c1a'))
)
$cwS = 9.0; $gap = 14; $rowCss = ''; $rowSvg = ''
for ($r = 0; $r -lt $rows.Count; $r++) {
    $chips = @(); $setW = 0
    foreach ($t in $rows[$r]) {
        $n = Len $t[0]; $w = 34 + $n * $cwS + 18
        $chips += , @($t[0], $t[1], $w, $n); $setW += $w + $gap
    }
    $copies = [int][math]::Ceiling(1100 / $setW) + 1
    $g = ''; $x = 0
    for ($c = 0; $c -lt $copies; $c++) {
        foreach ($ch in $chips) {
            $nm = $ch[0]; $col = $ch[1]; $w = $ch[2]; $n = $ch[3]
            $g += "<rect x='$x' y='0' width='$w' height='40' rx='20' fill='#1a0b33' stroke='$col' stroke-opacity='.65' stroke-width='1.5'/><circle cx='$($x + 20)' cy='20' r='5' fill='$col'/><text x='$($x + 34)' y='25' font-size='15' font-weight='600' fill='#f3e8ff' textLength='$([math]::Round($n * $cwS, 2))' lengthAdjust='spacing'>$nm</text>"
            $x += $w + $gap
        }
    }
    $dur = [math]::Round($setW / 55, 1)
    $from = if ($r % 2 -eq 0) { 0 } else { -$setW }
    $to = if ($r % 2 -eq 0) { -$setW } else { 0 }
    $rowCss += "@keyframes sc$r{from{transform:translateX(${from}px)}to{transform:translateX(${to}px)}}.sc$r{animation:sc$r ${dur}s linear infinite}`n"
    $rowSvg += "<g transform='translate(0,$(12 + $r * 54))'><g class='sc$r'>$g</g></g>"
}
$skills = @"
<svg xmlns='http://www.w3.org/2000/svg' width='1000' height='180' viewBox='0 0 1000 180' role='img' aria-label='Technologies'>
<defs>
<linearGradient id='fade' x1='0' x2='1' y1='0' y2='0'><stop offset='0' stop-color='#000'/><stop offset='.1' stop-color='#fff'/><stop offset='.9' stop-color='#fff'/><stop offset='1' stop-color='#000'/></linearGradient>
<mask id='m'><rect width='1000' height='180' fill='url(#fade)'/></mask>
</defs>
<style>
text{font-family:$MONO}
$rowCss
</style>
<g mask='url(#m)'>$rowSvg</g>
</svg>
"@
Save 'skills.svg' $skills

# =====================================================================
# TITLES + DIVIDER
# =====================================================================
function Title($file, $text) {
    $t = "[ $text ]"
    $n = Len $t; $tw = $n * 22; $x = [math]::Round((1000 - $tw) / 2, 1)
    $ux = [math]::Round(500 - $tw / 2 - 10, 1); $ue = [math]::Round(500 + $tw / 2 + 10, 1)
    $svg = @"
<svg xmlns='http://www.w3.org/2000/svg' width='1000' height='76' viewBox='0 0 1000 76' role='img' aria-label='$text'>
<defs>
<linearGradient id='g' gradientUnits='userSpaceOnUse' x1='$ux' y1='0' x2='$ue' y2='0' spreadMethod='repeat'><stop offset='0' stop-color='#c084fc'/><stop offset='.33' stop-color='#f472b6'/><stop offset='.66' stop-color='#22d3ee'/><stop offset='1' stop-color='#c084fc'/>
<animateTransform attributeName='gradientTransform' type='translate' from='0 0' to='$([math]::Round($ue - $ux, 1)) 0' dur='4s' repeatCount='indefinite'/></linearGradient>
<linearGradient id='ln' x1='0' x2='1'><stop offset='0' stop-color='#a855f7' stop-opacity='0'/><stop offset='.5' stop-color='#a855f7'/><stop offset='1' stop-color='#a855f7' stop-opacity='0'/></linearGradient>
<filter id='glow'><feGaussianBlur stdDeviation='4' result='b'/><feMerge><feMergeNode in='b'/><feMergeNode in='SourceGraphic'/></feMerge></filter>
</defs>
<style>
text{font-family:$MONO}
.t{animation:pop 4s ease-in-out infinite}
@keyframes pop{0%,100%{opacity:.92}50%{opacity:1}}
</style>
<g filter='url(#glow)'><text class='t' x='$x' y='42' font-size='30' font-weight='800' fill='url(#g)' textLength='$tw' lengthAdjust='spacing'>$t</text></g>
<rect x='$ux' y='56' width='$([math]::Round($ue - $ux, 1))' height='2' rx='1' fill='url(#ln)'/>
<circle r='4' fill='#fff' filter='url(#glow)'><animateMotion dur='3.2s' repeatCount='indefinite' path='M$ux,57 H$ue H$ux'/></circle>
</svg>
"@
    Save $file $svg
}
Title 'title-whoami.svg' 'WHOAMI'
Title 'title-stack.svg' 'STACK TECHNIQUE'
Title 'title-stats.svg' 'STATS GITHUB'
Title 'title-contact.svg' 'CONTACT'

$divider = @"
<svg xmlns='http://www.w3.org/2000/svg' width='1000' height='28' viewBox='0 0 1000 28' role='presentation'>
<defs>
<linearGradient id='ln' x1='0' x2='1'><stop offset='0' stop-color='#a855f7' stop-opacity='0'/><stop offset='.3' stop-color='#a855f7'/><stop offset='.5' stop-color='#22d3ee'/><stop offset='.7' stop-color='#f472b6'/><stop offset='1' stop-color='#f472b6' stop-opacity='0'/></linearGradient>
<filter id='glow'><feGaussianBlur stdDeviation='3' result='b'/><feMerge><feMergeNode in='b'/><feMergeNode in='SourceGraphic'/></feMerge></filter>
</defs>
<style>.d{transform-origin:500px 14px;animation:sp 3s ease-in-out infinite}@keyframes sp{0%,100%{transform:rotate(45deg) scale(1)}50%{transform:rotate(225deg) scale(1.35)}}</style>
<rect x='60' y='13' width='880' height='2' rx='1' fill='url(#ln)'/>
<rect class='d' x='494' y='8' width='12' height='12' fill='#fff' filter='url(#glow)'/>
<circle r='3.5' cy='14' fill='#fff' filter='url(#glow)'><animateMotion dur='4s' repeatCount='indefinite' path='M60,0 H940 H60'/></circle>
</svg>
"@
Save 'divider.svg' $divider

# =====================================================================
# BUTTONS
# =====================================================================
function Button($file, $label, $c1, $c2, $c3, $icon, $tw) {
    $svg = @"
<svg xmlns='http://www.w3.org/2000/svg' width='250' height='64' viewBox='0 0 250 64' role='img' aria-label='$label'>
<defs>
<linearGradient id='b' gradientUnits='userSpaceOnUse' x1='0' y1='0' x2='250' y2='0'><stop offset='0' stop-color='$c1'/><stop offset='.5' stop-color='$c2'/><stop offset='1' stop-color='$c3'/>
<animateTransform attributeName='gradientTransform' type='rotate' from='0 125 32' to='360 125 32' dur='4s' repeatCount='indefinite'/></linearGradient>
<clipPath id='pill'><rect x='4' y='4' width='242' height='56' rx='28'/></clipPath>
<filter id='gl' x='-20%' y='-40%' width='140%' height='180%'><feGaussianBlur stdDeviation='6'/></filter>
</defs>
<style>
text{font-family:$MONO}
.pulse{animation:pu 2.4s ease-in-out infinite}@keyframes pu{0%,100%{opacity:.25}50%{opacity:.7}}
.shine{animation:sh 3.2s ease-in-out infinite}@keyframes sh{0%{transform:translateX(-120px) skewX(-20deg)}60%,100%{transform:translateX(300px) skewX(-20deg)}}
</style>
<rect class='pulse' x='4' y='4' width='242' height='56' rx='28' fill='none' stroke='url(#b)' stroke-width='6' filter='url(#gl)'/>
<rect x='4' y='4' width='242' height='56' rx='28' fill='#130725' stroke='url(#b)' stroke-width='2.5'/>
<g clip-path='url(#pill)'><rect class='shine' x='0' y='0' width='40' height='64' fill='#ffffff' fill-opacity='.14'/></g>
$icon
<text x='$(78)' y='39' font-size='20' font-weight='800' fill='#f3e8ff' textLength='$tw' lengthAdjust='spacing'>$label</text>
</svg>
"@
    Save $file $svg
}
$iconIn = "<rect x='30' y='17' width='30' height='30' rx='6' fill='#0a66c2'/><text x='45' y='40' text-anchor='middle' font-size='20' font-weight='800' fill='#fff'>in</text>"
$iconMail = "<rect x='30' y='20' width='32' height='24' rx='4' fill='none' stroke='#f3e8ff' stroke-width='2.4'/><path d='M31,22 L46,34 L61,22' fill='none' stroke='#f3e8ff' stroke-width='2.4' stroke-linejoin='round'/>"
Button 'btn-linkedin.svg' 'LinkedIn' '#0a66c2' '#38bdf8' '#a855f7' $iconIn 104
Button 'btn-email.svg' 'E-mail' '#f472b6' '#a855f7' '#22d3ee' $iconMail 78

# =====================================================================
# FOOTER
# =====================================================================
$ft = 'MERCI DE VOTRE VISITE'
$ftw = (Len $ft) * 20
$footer = @"
<svg xmlns='http://www.w3.org/2000/svg' width='1000' height='170' viewBox='0 0 1000 170' role='img' aria-label='Merci de votre visite'>
<defs>
<linearGradient id='bg' x1='0' y1='0' x2='1' y2='1'><stop offset='0' stop-color='#2e0b5c'/><stop offset='1' stop-color='#07010f'/></linearGradient>
<linearGradient id='tg' gradientUnits='userSpaceOnUse' x1='250' y1='0' x2='750' y2='0' spreadMethod='repeat'><stop offset='0' stop-color='#c084fc'/><stop offset='.5' stop-color='#22d3ee'/><stop offset='1' stop-color='#c084fc'/>
<animateTransform attributeName='gradientTransform' type='translate' from='0 0' to='500 0' dur='5s' repeatCount='indefinite'/></linearGradient>
<clipPath id='card'><rect width='1000' height='170' rx='26'/></clipPath>
<filter id='glow'><feGaussianBlur stdDeviation='5' result='b'/><feMerge><feMergeNode in='b'/><feMergeNode in='SourceGraphic'/></feMerge></filter>
</defs>
<style>
text{font-family:$MONO}
$COMMON_CSS
.hb{transform-origin:500px 108px;transform-box:view-box;animation:hb 1.6s ease-in-out infinite}@keyframes hb{0%,100%{transform:scale(1)}15%{transform:scale(1.25)}30%{transform:scale(1)}45%{transform:scale(1.18)}}
</style>
<g clip-path='url(#card)'>
<rect width='1000' height='170' fill='url(#bg)'/>
$(Stars 1000 170 40 21)
$(Orbs 1000 120 170 10 5)
$(Wave 118 10 500 '#6A0DAD' 0.5 13 $true 60)
$(Wave 128 8 250 '#a855f7' 0.35 9 $false 50)
$(Wave 138 6 200 '#22d3ee' 0.2 7 $true 40)
<g filter='url(#glow)'><text x='$((1000 - $ftw) / 2)' y='62' font-size='30' font-weight='800' fill='url(#tg)' textLength='$ftw' lengthAdjust='spacing'>$ft</text></g>
<text x='500' y='86' text-anchor='middle' font-size='16' fill='#e9d5ff' fill-opacity='.85' textLength='330' lengthAdjust='spacing'>Fait avec passion au Havre &#183; Nailt</text>
<g transform='translate(0,8)'><path class='hb' d='M500,124 C492,114 480,110 480,101 C480,95 485,92 490,92 C495,92 499,95 500,99 C501,95 505,92 510,92 C515,92 520,95 520,101 C520,110 508,114 500,124 Z' fill='#f472b6'/></g>
</g>
<rect x='1' y='1' width='998' height='168' rx='25' fill='none' stroke='#a855f7' stroke-opacity='.55' stroke-width='2'/>
</svg>
"@
Save 'footer.svg' $footer

Get-ChildItem $out | Select-Object Name, Length

<p align="center">
  <img src="https://capsule-render.vercel.app/api?type=waving&color=6A0DAD&height=200&section=header&text=Jonathan%20LECLERC&fontSize=50&fontColor=ffffff&animation=fadeIn" />
</p>

<h2 align="center">
  <img src="https://readme-typing-svg.herokuapp.com/?font=Righteous&size=35&center=true&vCenter=true&width=500&height=70&duration=4000&lines=Hi+There!+👋;+I'm+Nailt!" />
</h2>

<p align="center">
  <b>Le Havre, France</b> <br>
  <i>Actuellement en BUT informatique en 3e année </i> <br>
  <i>Je suis également Apprenti dans l'entreprise IES Ingénierie, Port Jérôme-Sur-Seine pour l'année 2026/2027</i>
</p>

---

### Langages et Logiciels

<p align="center">
  <img src="https://skillicons.dev/icons?i=js,html,css,bootstrap,php,mysql,postgres,java,cs,py,c,bash,linux,docker,git,unity,vscode&theme=dark" />
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Autodesk_Fusion_360-EA2808?style=flat-square&logo=autodesk&logoColor=white" />
</p>

- 🌐 **Web :** `HTML5`, `CSS3`, `JavaScript`, `Bootstrap`, `PHP`, `Laravel`, `Tailwind`, `Twig`
- 🗄️ **Bases de données :** `MySQL`, `PostgreSQL`, `PlpgSql`
- ☕ **Applicatif & Mobile :** `Java`, `Android Studio`, `C#`, `Python`, `C`
- 🛠️ **Système & Outils :** `Linux`, `Git`, `Docker`, `Bash`, `VS Code`, `Laragon`
- 🎨 **Création & Modélisation :** `Fusion 360`, `Unity`, `Scratch`

---

### Contact

<p align="center">
  <a href="https://www.linkedin.com/in/jonathanleclerc2006/">
    <img src="https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white" />
  </a>
  <a href="mailto:jonathan10leclerc@gmail.com">
    <img src="https://img.shields.io/badge/Email-D14836?style=for-the-badge&logo=gmail&logoColor=white" />
  </a>
</p>
