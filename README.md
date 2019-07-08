# Assembly-Rhythm-Game

[![Video Label](http://img.youtube.com/vi/fmd-7fw2pIU/0.jpg)](https://youtu.be/fmd-7fw2pIU) <br>
이미지을 클릭하시면 이동합니다.<br>
> #게임 이름 : 완벽한 걸로<br>
> #피아노 타일 모작<br>
> #제작자 : 이창훈, 구병조<br>
> #제작 기간 : 1달<br>
> #제작 언어 : 어셈블리어 (PIC 마이컴)<br>
> #개발 플랫폼 : MPLAB_IDE_8_83<br>
> #기반 기종 : PIC16F876A<br>
> #사용된 헤더 : P16F876A.inc<br>

## Display
<pre><code>DISP
	INCF	INC,F
	BTFSS	INC,2
	GOTO	DISPA
	GOTO	DISPB
DISPA
	BTFSS	INC,1
	GOTO	DISPAA
	GOTO	DISPAB

DISPBB
	BTFSS	INC,0
	GOTO	DISP4
	GOTO	DISPLED
ST
	BCF	PORTC,7
	BCF	PORTA,1
	BCF	PORTC,1
	BCF	PORTA,0
	RETURN
DISPLED
	BSF	PORTB,1
	BSF	PORTA,3
	BSF	PORTA,2
	BSF	PORTB,2
	BSF	PORTB,7
	BTFSC	BT,0
	CALL	LED1
	BTFSC	BT,1
	CALL	LED2
	BTFSC	BT,2
	CALL	LED3
	BCF PORTC,5
	BCF PORTC,6
	BCF PORTC,7
	RETURN
DISP1	
	BCF	PORTB,	7
	CALL	ST
	BTFSC	NOTEA,4
	BSF	PORTC,	7
	BTFSC	NOTEB,4
	BSF	PORTA,	1	
	BTFSC	NOTEC,4
	BSF	PORTC,	1
	BTFSC	ER,3
	BSF	PORTA,0
	BTFSC	ER,4
	CALL	FF
	BCF	PORTA,3
	RETURN</code></pre>
키트의 회로상 7Seg의 

