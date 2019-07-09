# Assembly-Rhythm-Game
## 개요
[![Video Label](http://img.youtube.com/vi/fmd-7fw2pIU/0.jpg)](https://youtu.be/fmd-7fw2pIU) <br>
이미지을 클릭하시면 영상으로 이동합니다.<br>
> #게임 이름 : 완벽한 걸로<br>
> #피아노 타일 모작<br>
> #제작자 : 이창훈, 구병조<br>
> #제작 기간 : 1달<br>
> #제작 언어 : 어셈블리어 (PIC 마이컴)<br>
> #개발 플랫폼 : MPLAB_IDE_8_83<br>
> #기반 기종 : PIC16F876A<br>
> #사용된 헤더 : P16F876A.inc<br>
## 코드 분석
### Interupt
<pre><code>GOTO	START_UP
	ORG	04H
	;	ISR	시작	번지
	MOVWF	W_TEMP	;	현재	사용되고	있는	W	REG.	저장
	SWAPF	STATUS,	W
	MOVWF	STATUS_TEMP
	CALL	DISP	;	DISPLAY	부	프로그램
	SWAPF	STATUS_TEMP,	W
	MOVWF	STATUS
	SWAPF	W_TEMP,	F
	SWAPF	W_TEMP,	W
	BCF	INTCON,	2
	RETFIE</code></pre>
<br>
인터럽트가 발생하면 우선 Working Resister에 저장된 값이 인터럽트 전후에 훼손되는 것을 막는 것이 중요하다.<br>
원하는 임무를 수행하는 전 후에 Working Resister를 Carry에 영향주지 않으면서 저장하기 위한 부분이 있음을 볼 수 있다.

### Display
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
.	BSF	PORTA,2
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
키트의 회로상 7Seg의 a부분과 LED3의 출력 포트가 같으므로, 두개의 라이트를 동시에 원하는 대로 조절할 수 없다.<br>
따라서 7seg 4개의 digit의 노트 위치와 실패 횟수를 파악하는 코드와, 그 사이사이에 있는 LED 조절 코드가 분리되어야 하고 한 쪽을 조절할 때에 다른 한 쪽을 off시켜야 한다.

### 시작
<pre><code>START_UP
	BSF	STATUS,	RP0	;	RAM	BANK	1	선택
	MOVLW	B'00000000'	;	PORT	I/O	선택
	MOVWF	TRISA
	MOVLW	B'00111000'	;	PORT	I/O	선택
	MOVWF	TRISB
	MOVLW	B'00000000'	;	PORT	I/O	선택
	MOVWF	TRISC
	MOVLW	B'00000111'
	MOVWF	ADCON1
;	INTERRUPT	시간	설정	---	2.048	msec	주기
	MOVLW	B'00000000'	;	2.048	msec
	MOVWF	OPTION_REG
	BCF	STATUS,	RP0	;	RAM	BANK	0	선택
	CALL START
	BSF	INTCON,	5	;	TIMER	INTERRUPT	ENABLE
	BSF	INTCON,	7	;	GLOBAL	INTERRUPT	ENABLE
	GOTO	MAIN_ST</code></pre><br>
어셈블리에서는 메모리 사용에 있어 Bank 역시 수동으로 이동해야 하므로, 포트 입출력을 설정하는 Bank01의 특수 기능 레지스터에 접근하기 위해 별도의 코드가 필요하다<br>
입출력 설정이 완료 된 후 카운트 다운 함수인 Start를 수행한 이후 Interrupt를 허용함으로써 게임을 진행시킨다.<br>
### 카운트 다운
<pre><code>START
	BSF	PORTB,1
	BSF	PORTA,3
	BSF	PORTA,2
	BSF	PORTB,2
	MOVLW	B'11100010'
	MOVWF	PORTC
	BCF	PORTA,0
	BSF	PORTA,1
	BCF PORTB,1
	CALL DELAY
	BSF PORTB,1
	MOVLW	B'11000011'
	MOVWF	PORTC
	BCF	PORTA,0
	BSF	PORTA,1
	BCF PORTB,2
	CALL DELAY
	BSF PORTB,2
	MOVLW	B'01100000'
	MOVWF	PORTC
	BCF	PORTA,0
	BCF	PORTA,1
	BCF PORTA,2
	CALL DELAY
	RETURN</code></pre>
7Seg에 사용자가 게임 시작에 대비할 수 있도록 카운트 다운을 표기한다.
### 속도 조절
<pre><code>M_LOOP
	BTFSS	SPEED,0
	GOTO	SPEEDA
	GOTO	SPEEDB
SPEEDA
	MOVLW	.120
	SUBWF	INT_CNT,W
	BTFSS	STATUS,	Z
	GOTO	XLOOP
	GOTO	CK_LOOP</code></pre>
리듬게임에서 노트가 다가오는 속도는 난이도와 직결된다.<br>
이 코드에서는 인터럽트가 시행된 횟수에 따라 조절했다.
### 떨어지는 노트
<pre><code>CK_LOOP
	CLRF	INT_CNT
	ADDLW	0
	RRF	NOTEAS,F
	RRF	NOTEA,F
	ADDLW	0
	RRF	NOTEBS,F
	RRF	NOTEB,F
	ADDLW	0
	RRF	NOTECS,F
	RRF	NOTEC,F
	ADDLW	0</code></pre>
노트의 배열이 저장된 레지스터의 값을 1칸씩 비트이동시켜 노트의 위치를 바꾼다.<br>
1라인 당 2개의 레지스터를 할당한 것은 8비트 이상의 노트 대기열이 존재하기를 원했기 때문이다.<br>
노트의 위치를 이동하면서, 예상치 못한 Carry의 발생을 막기 위해 레지스터에 0을 더하는 코드가 사이사이 끼어있다.
### 에러 체크
<pre><code>XLOOP
	INCF	TIME,F
	BTFSC	NOTEA,0
	CALL	ERR
	BTFSC	NOTEB,0
	CALL	ERR	
	BTFSC	NOTEC,0
	CALL	ERR
	GOTO	M_LOOP	</code></pre>
각 라인별 레지스터에 7Seg 출력지점을 넘어선(미처 없애지 못한) 노트가 있는지 체크하고, 있다면 에러함수를 호출한다.
### 에러 추가
<pre><code>ERR
	CALL	RANDOMERR
	BCF	NOTEA,0
	BCF	NOTEB,0
	BCF	NOTEC,0
	BTFSC	ER,	3
	BSF	ER,4
	BTFSC	ER,	4
	BCF PORTA,4
	BTFSC	ER,	2
	BSF	ER,3
	BTFSC	ER,	1
	BSF	ER,2
	BTFSC	ER,	0
	BSF	ER,1
	BSF	ER,0
	RETURN</code></pre>
에러가 일어난 횟수를 단순히 데이터에 1씩 더하는 방식을 사용한다면, 데이터 기록은 쉽지만 표기를 위해 불러올 때의 어려움이 있다.<br>
따라서 위치별로 에러 카운트를 저장함으로서 별도의 계산없이 비트체크만으로 에러의 횟수를 표기하기 위해 해당 방식을 사용했다.
### 버튼 인식
<pre><code>BTN
	MOVF	PORTB,W
	MOVWF	DN
	ADDLW	0
	RRF	DN,F
	RRF	DN,F
	RRF	DN,W
	ANDLW	B'00000111'
	CALL	LU
	RETURN
LU
	ADDWF	PCL,F
	GOTO	S0
	GOTO	S1
	GOTO	S2
	GOTO	S3
	GOTO	S4
	GOTO	S5
	GOTO	S6
	GOTO	S7</code></pre>
3개의 Push스위치로부터 받은 PORTB레지스터 정보를 마스킹한 다음 룩업 테이블을 이용해 내용을 분기하여 각종 기능 구현을 위한 틀을 제작하였다.<br>
Program Counter을 조절하여 원하는 위치에 함수를 실행한다.
### 버튼 조작을 통한 각종 기능들
#### 재시작
<pre><code>S0	;3개의 버튼 동시에 누름
	CALL REGAME
	RETURN</code></pre>
단순히 시작부의 위치로 PC만 이동한다.
#### 속도 증가
<pre><code>
S4	;오른쪽 2개의 버튼 동시에 누름
	CALL UP
	RETURN
UP
	BTFSS SPEED,0
	INCF SPEED,F
	CLRF INT_CNT
	RETURN</code></pre>
인터럽트 횟수 초기화 후 SPEED 플래그 설정으로 필요 인터럽트 횟수 감소
#### 속도 감소
<pre><code>
S1	;왼쪽 2개의 버튼 동시에 누름
	CALL DOWN
	RETURN
DOWN
	BTFSC SPEED,0
	DECF SPEED,F
	CLRF INT_CNT
	RETURN</code></pre>
인터럽트 횟수 초기화 후 SPEED 플래그 설정으로 필요 인터럽트 횟수 증가
#### 노트 제거
<pre><code>
S3	;가장 왼쪽 버튼 누름
	BTFSC	NOTEC,1
	CALL	RANDOM3
	BCF	NOTEC,1
	RETURN</code></pre>
RANDOM3
	BCF PORTA,4
	BCF	NOTEA,6
	BCF	NOTEB,6
	BCF	NOTEC,6
	BTFSS	TIME,0
	GOTO	SA
	GOTO	SB
SA
	BSF	NOTEA,6
	BSF NOTEB,7
	RETURN
SB
	BSF	NOTEB,6
	BSF NOTEC,7
	RETURN</code></pre>
	눌린 버튼의 최하단 노트 클리어 이후 랜덤으로 새로운 노트 대기열에 추가
#### 일시정지
<pre><code>
S2	;양쪽 버튼 동시에 누름
	CLRF INT_CNT
	RETURN</code></pre>
