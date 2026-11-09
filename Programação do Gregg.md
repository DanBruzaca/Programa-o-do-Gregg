// ==========================================
// MOTOR Y - EIXO Y
// ==========================================

#define RPWM_Y 9
#define LPWM_Y 10

#define BOTAO_SUBIR 2
#define BOTAO_DESCER 3

#define VELOCIDADE_SUBIR 40
#define VELOCIDADE_DESCER 115


// ==========================================
// MOTOR X - EIXO X
// ==========================================

#define RPWM_X 5
#define LPWM_X 6

#define BOTAO_INICIAR 4
#define BOTAO_VELOCIDADE 7


// ==========================================
// PARADA GERAL
// ==========================================

#define BOTAO_PARADA 8


// ==========================================
// LEDs
// ==========================================

#define LED_VERDE 11
#define LED_VERMELHO 12


// ==========================================
// VELOCIDADES DO MOTOR X
// ==========================================

int velocidades[] = {100, 115, 180};

int nivelVelocidade = 0;


// ==========================================
// ESTADOS DO SISTEMA
// ==========================================

bool motorXLigado = false;
bool sistemaBloqueado = false;

bool modoAutomatico = false;


// ==========================================
// CONTROLE DOS BOTÕES
// ==========================================

bool ultimoBotaoIniciar = HIGH;
bool ultimoBotaoVelocidade = HIGH;
bool ultimoBotaoParada = HIGH;


// ==========================================
// ATIVAÇÃO DO MODO AUTOMÁTICO
// ==========================================

unsigned long inicioCombinacao = 0;

bool combinacaoAtiva = false;
bool automaticoAtivado = false;


// ==========================================
// PISCADA DOS LEDs
// ==========================================

bool piscando = false;
bool estadoPiscada = false;

int quantidadePiscadas = 0;

unsigned long tempoPiscada = 0;


// ==========================================
// CONTROLE DO MODO AUTOMÁTICO
// ==========================================

int etapaAutomatica = 0;

unsigned long tempoEtapa = 0;


// ==========================================
// SETUP
// ==========================================

void setup() {

  // ----------------------------------------
  // MOTOR Y
  // ----------------------------------------

  pinMode(RPWM_Y, OUTPUT);
  pinMode(LPWM_Y, OUTPUT);

  pinMode(BOTAO_SUBIR, INPUT_PULLUP);
  pinMode(BOTAO_DESCER, INPUT_PULLUP);


  // ----------------------------------------
  // MOTOR X
  // ----------------------------------------

  pinMode(RPWM_X, OUTPUT);
  pinMode(LPWM_X, OUTPUT);

  pinMode(BOTAO_INICIAR, INPUT_PULLUP);
  pinMode(BOTAO_VELOCIDADE, INPUT_PULLUP);


  // ----------------------------------------
  // PARADA GERAL
  // ----------------------------------------

  pinMode(BOTAO_PARADA, INPUT_PULLUP);


  // ----------------------------------------
  // LEDs
  // ----------------------------------------

  pinMode(LED_VERDE, OUTPUT);
  pinMode(LED_VERMELHO, OUTPUT);


  // ----------------------------------------
  // SISTEMA COMEÇA LIBERADO
  // ----------------------------------------

  digitalWrite(LED_VERDE, HIGH);
  digitalWrite(LED_VERMELHO, LOW);


  // ----------------------------------------
  // MOTORES DESLIGADOS
  // ----------------------------------------

  analogWrite(RPWM_Y, 0);
  analogWrite(LPWM_Y, 0);

  analogWrite(RPWM_X, 0);
  analogWrite(LPWM_X, 0);
}


// ==========================================
// LOOP PRINCIPAL
// ==========================================

void loop() {


  // ========================================
  // PARADA GERAL - PRIORIDADE MÁXIMA
  // ========================================

  bool estadoParada = digitalRead(BOTAO_PARADA);


  if (ultimoBotaoParada == HIGH && estadoParada == LOW) {

    // Bloqueia o sistema
    sistemaBloqueado = true;

    // Reseta todos os modos
    modoAutomatico = false;
    automaticoAtivado = false;

    motorXLigado = false;

    combinacaoAtiva = false;
    piscando = false;

    etapaAutomatica = 0;


    // --------------------------------------
    // PARA MOTOR Y
    // --------------------------------------

    analogWrite(RPWM_Y, 0);
    analogWrite(LPWM_Y, 0);


    // --------------------------------------
    // PARA MOTOR X
    // --------------------------------------

    analogWrite(RPWM_X, 0);
    analogWrite(LPWM_X, 0);


    // --------------------------------------
    // LEDS
    // --------------------------------------

    digitalWrite(LED_VERDE, LOW);
    digitalWrite(LED_VERMELHO, HIGH);
  }


  ultimoBotaoParada = estadoParada;


  // ========================================
  // SISTEMA BLOQUEADO
  // ========================================

  if (sistemaBloqueado) {

    // Garante motores parados

    analogWrite(RPWM_Y, 0);
    analogWrite(LPWM_Y, 0);

    analogWrite(RPWM_X, 0);
    analogWrite(LPWM_X, 0);


    // --------------------------------------
    // BOTÃO INICIAR PARA REARMAR
    // --------------------------------------

    bool estadoBotaoIniciar = digitalRead(BOTAO_INICIAR);


    if (ultimoBotaoIniciar == HIGH &&
        estadoBotaoIniciar == LOW) {


      // Desbloqueia
      sistemaBloqueado = false;


      // Reseta os modos
      modoAutomatico = false;
      automaticoAtivado = false;

      motorXLigado = false;

      combinacaoAtiva = false;
      piscando = false;

      estadoPiscada = false;

      quantidadePiscadas = 0;

      etapaAutomatica = 0;


      // LEDs
      digitalWrite(LED_VERDE, HIGH);
      digitalWrite(LED_VERMELHO, LOW);


      // Garante motores parados
      analogWrite(RPWM_Y, 0);
      analogWrite(LPWM_Y, 0);

      analogWrite(RPWM_X, 0);
      analogWrite(LPWM_X, 0);


      delay(100);
    }


    ultimoBotaoIniciar = estadoBotaoIniciar;

    return;
  }


  // ========================================
  // LEITURA DOS BOTÕES D4 E D7
  // ========================================

  bool botaoIniciar = digitalRead(BOTAO_INICIAR);
  bool botaoVelocidade = digitalRead(BOTAO_VELOCIDADE);


  // ========================================
  // ATIVAÇÃO DO MODO AUTOMÁTICO
  // D4 + D7 POR 2 SEGUNDOS
  // ========================================

  if (botaoIniciar == LOW &&
      botaoVelocidade == LOW) {


    if (!combinacaoAtiva) {

      combinacaoAtiva = true;

      inicioCombinacao = millis();
    }


    // Depois de 2 segundos
    if (!automaticoAtivado &&
        millis() - inicioCombinacao >= 2000) {


      automaticoAtivado = true;

      modoAutomatico = true;


      // ------------------------------------
      // INICIA A SEQUÊNCIA
      // ------------------------------------

      etapaAutomatica = 0;

      tempoEtapa = millis();


      // ------------------------------------
      // INICIA AS 5 PISCADAS
      // ------------------------------------

      piscando = true;

      estadoPiscada = true;

      quantidadePiscadas = 0;

      tempoPiscada = millis();


      // Liga os dois LEDs
      digitalWrite(LED_VERDE, HIGH);
      digitalWrite(LED_VERMELHO, HIGH);
    }

  }

  else {

    // Botões foram soltos
    combinacaoAtiva = false;
  }


  // ========================================
  // PISCAR LEDs 5 VEZES
  // ========================================

  if (piscando) {


    if (millis() - tempoPiscada >= 300) {

      tempoPiscada = millis();


      // Inverte estado dos LEDs
      estadoPiscada = !estadoPiscada;


      digitalWrite(LED_VERDE, estadoPiscada);
      digitalWrite(LED_VERMELHO, estadoPiscada);


      // Conta uma piscada quando volta para LOW
      if (!estadoPiscada) {

        quantidadePiscadas++;


        if (quantidadePiscadas >= 5) {

          piscando = false;


          // --------------------------------
          // FINAL DAS PISCADAS
          // --------------------------------

          digitalWrite(LED_VERDE, HIGH);
          digitalWrite(LED_VERMELHO, LOW);


          // Reinicia o tempo da sequência
          tempoEtapa = millis();

          etapaAutomatica = 0;
        }
      }
    }


    // Não executa motores durante as piscadas
    return;
  }


  // ========================================
  // MODO AUTOMÁTICO
  // ========================================

  if (modoAutomatico) {


    unsigned long agora = millis();


    // ======================================
    // ETAPA 0
    // MOTOR X - 1 SEGUNDO
    // ======================================

    if (etapaAutomatica == 0) {


      // Motor X ligado
      analogWrite(RPWM_X, velocidades[1]);
      analogWrite(LPWM_X, 0);


      // Motor Y parado
      analogWrite(RPWM_Y, 0);
      analogWrite(LPWM_Y, 0);


      if (agora - tempoEtapa >= 1000) {

        etapaAutomatica = 1;

        tempoEtapa = agora;
      }
    }


    // ======================================
    // ETAPA 1
    // MOTOR Y SOBE - 1 SEGUNDO
    // ======================================

    else if (etapaAutomatica == 1) {


      // Motor X ligado
      analogWrite(RPWM_X, velocidades[1]);
      analogWrite(LPWM_X, 0);


      // Motor Y subindo
      analogWrite(RPWM_Y, VELOCIDADE_SUBIR);
      analogWrite(LPWM_Y, 0);


      if (agora - tempoEtapa >= 1088) {

        etapaAutomatica = 2;

        tempoEtapa = agora;
      }
    }


    // ======================================
    // ETAPA 2
    // MOTOR Y PARADO - 1 SEGUNDO
    // ======================================

    else if (etapaAutomatica == 2) {


      // Motor X continua ligado
      analogWrite(RPWM_X, velocidades[1]);
      analogWrite(LPWM_X, 0);


      // Motor Y parado
      analogWrite(RPWM_Y, 0);
      analogWrite(LPWM_Y, 0);


      if (agora - tempoEtapa >= 5000) {

        etapaAutomatica = 3;

        tempoEtapa = agora;
      }
    }


    // ======================================
    // ETAPA 3
    // MOTOR Y DESCE - 1 SEGUNDO
    // ======================================

    else if (etapaAutomatica == 3) {


      // Motor X ligado
      analogWrite(RPWM_X, velocidades[1]);
      analogWrite(LPWM_X, 0);


      // Motor Y descendo
      analogWrite(RPWM_Y, 0);
      analogWrite(LPWM_Y, VELOCIDADE_DESCER);


      if (agora - tempoEtapa >= 1300) {

        etapaAutomatica = 4;

        tempoEtapa = agora;
      }
    }


    // ======================================
    // ETAPA 4
    // MOTOR Y PARADO - 1 SEGUNDO
    // ======================================

    else if (etapaAutomatica == 4) {


      // Motor X continua ligado
      analogWrite(RPWM_X, velocidades[1]);
      analogWrite(LPWM_X, 0);


      // Motor Y parado
      analogWrite(RPWM_Y, 0);
      analogWrite(LPWM_Y, 0);


      if (agora - tempoEtapa >= 1000) {

        // ----------------------------------
        // VOLTA PARA O INÍCIO
        // ----------------------------------

        etapaAutomatica = 0;

        tempoEtapa = agora;
      }
    }


    // Não executa o modo manual
    return;
  }


  // ========================================
  // MODO MANUAL
  // ========================================


  // ========================================
  // EIXO Y - SUBIR
  // ========================================

  if (digitalRead(BOTAO_SUBIR) == LOW) {

    analogWrite(RPWM_Y, VELOCIDADE_SUBIR);
    analogWrite(LPWM_Y, 0);
  }


  // ========================================
  // EIXO Y - DESCER
  // ========================================

  else if (digitalRead(BOTAO_DESCER) == LOW) {

    analogWrite(RPWM_Y, 0);
    analogWrite(LPWM_Y, VELOCIDADE_DESCER);
  }


  // ========================================
  // EIXO Y - PARADO
  // ========================================

  else {

    analogWrite(RPWM_Y, 0);
    analogWrite(LPWM_Y, 0);
  }


  // ========================================
  // BOTÃO INICIAR - MOTOR X
  // ========================================

  bool estadoBotaoIniciar = digitalRead(BOTAO_INICIAR);


  if (ultimoBotaoIniciar == HIGH &&
      estadoBotaoIniciar == LOW) {

    motorXLigado = !motorXLigado;

    delay(50);
  }


  ultimoBotaoIniciar = estadoBotaoIniciar;


  // ========================================
  // BOTÃO VELOCIDADE
  // ========================================

  bool estadoBotaoVelocidade =
    digitalRead(BOTAO_VELOCIDADE);


  if (ultimoBotaoVelocidade == HIGH &&
      estadoBotaoVelocidade == LOW) {


    nivelVelocidade++;


    if (nivelVelocidade > 2) {

      nivelVelocidade = 0;
    }


    delay(50);
  }


  ultimoBotaoVelocidade = estadoBotaoVelocidade;


  // ========================================
  // MOTOR X - MODO MANUAL
  // ========================================

  if (motorXLigado) {

    analogWrite(RPWM_X, velocidades[nivelVelocidade]);
    analogWrite(LPWM_X, 0);
  }

  else {

    analogWrite(RPWM_X, 0);
    analogWrite(LPWM_X, 0);
  }
}
