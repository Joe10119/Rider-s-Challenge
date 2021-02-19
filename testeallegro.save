#include <allegro5/allegro.h>
#include <allegro5/allegro_native_dialog.h>
#include <allegro5/allegro_image.h>
#include <allegro5/allegro_font.h>
#include <allegro5/allegro_ttf.h>
#include <allegro5/allegro_audio.h>
#include <allegro5/allegro_acodec.h>
#include <stdio.h>
#include <string.h>

#define BACKGROUND_ARQUIVO "rua.png"
#define CARROVERM "carrovermelho.png"
#define CARROAZUL "carroazul.png"
#define FPS 60.0
#define LARGURA 360
#define ALTURA 720

//Declarção de todos assets
ALLEGRO_DISPLAY *display = NULL;
ALLEGRO_BITMAP *background = NULL;
ALLEGRO_BITMAP *personagem = NULL;
ALLEGRO_BITMAP *carroverm = NULL;
ALLEGRO_BITMAP *carroazul = NULL;
ALLEGRO_BITMAP *historia = NULL;
ALLEGRO_BITMAP *fundomenu = NULL;
ALLEGRO_BITMAP *gameover = NULL;
ALLEGRO_BITMAP *fundorank = NULL;
ALLEGRO_BITMAP *todasmotos = NULL;
ALLEGRO_BITMAP *textorank = NULL;
ALLEGRO_BITMAP *iconebmp = NULL;
ALLEGRO_SAMPLE *one = NULL;
ALLEGRO_SAMPLE_ID *oneid = NULL;
ALLEGRO_SAMPLE *musicamain = NULL;
ALLEGRO_SAMPLE *crashsom = NULL;
ALLEGRO_SAMPLE *partidamoto = NULL;
ALLEGRO_SAMPLE *logovoz = NULL;
ALLEGRO_EVENT_QUEUE *queue = NULL;
ALLEGRO_FONT *fonte = NULL;
ALLEGRO_FONT *fontemenor = NULL;
ALLEGRO_TIMER *timer = NULL;
FILE *scoreboard = NULL;
char str[21];
int motoselected = 0;

//Struct jogador para gravação no arquivo
struct playerst {
    int pontos;
    char nome[21];
};

typedef struct playerst Player;

//Gera uma mensagem de erro para identificação de exeções na inicialização
void error_msg(char *text){
	al_show_native_message_box(NULL,"ERRO",
		"Erro:",
		text,NULL,ALLEGRO_MESSAGEBOX_ERROR);
}

//Gera um número aleatório dentro de um intervalo, é usada para determinar a posição dos carros no eixo x
int random(int min, int max){
   return min + rand() / (RAND_MAX / (max - min + 1) + 1);
}

//inicializa todos módulos
int inicializacao(){
    if (!al_init()){
        error_msg("Falha ao inicializar.");
        return 0;
    }

    timer = al_create_timer(1.0 / FPS);
    if(!timer) {
        error_msg("Falha ao criar temporizador");
        return 0;
    }

    al_init_font_addon();

    if (!al_init_ttf_addon()){
        error_msg("Falha ao inicializar add-on allegro_ttf");
        return 0;
    }

    if (!al_init_image_addon()){
        error_msg("Falha ao inicializar add-on allegro_image");
        return 0;
    }

    if(!al_install_audio()){
      error_msg("Falha ao inicializar add-on audio");
      return 0;
    }

    if(!al_init_acodec_addon()){
      error_msg("Falha ao inicializar add-on acodec");
      return 0;
    }

    if (!al_reserve_samples(2)){
      error_msg("Falha ao reservar samples");
      return 0;
    }

    if (!al_install_keyboard()){
        error_msg("Falha ao inicializar o teclado");
        return 0;
    }

    display = al_create_display(LARGURA, ALTURA);
    if (!display){
        error_msg("Falha ao criar janela");
        return 0;
    }

    iconebmp = al_load_bitmap("icone.png");
    al_set_window_title(display, "Riders Challenge");
    al_set_display_icon(display, iconebmp);
    fonte = al_load_font("algerian.ttf", 36, 0);
    fontemenor = al_load_font("algerian.ttf", 24, 0);
    if (!fonte){
        error_msg("Falha ao carregar \"arial.ttf\"");
        al_destroy_display(display);
        al_destroy_timer(timer);
        return 0;
    }
    if (!fontemenor){
        error_msg("Falha ao carregar \"arial.ttf\"");
        al_destroy_display(display);
        al_destroy_timer(timer);
        return 0;
    }

    queue = al_create_event_queue();
    if (!queue){
        error_msg("Falha ao criar fila de eventos");
        al_destroy_display(display);
        al_destroy_timer(timer);
        return 0;
    }

    //Registra fontes de eventos, display, timer e teclado
    al_register_event_source(queue, al_get_keyboard_event_source());
    al_register_event_source(queue, al_get_display_event_source(display));
    al_register_event_source(queue, al_get_timer_event_source(timer));

    return 1;
}

//Função para leitura do nome dos jogadores
void manipular_entrada(ALLEGRO_EVENT evento)
{
    if (evento.type == ALLEGRO_EVENT_KEY_CHAR)
    {
        if (strlen(str) <= 20)
        {
            char temp[] = {evento.keyboard.unichar, '\0'};
            if (evento.keyboard.unichar == ' ')
            {
                strcat(str, temp);
            }
            else if (evento.keyboard.unichar >= '0' &&
                     evento.keyboard.unichar <= '9')
            {
                strcat(str, temp);
            }
            else if (evento.keyboard.unichar >= 'A' &&
                     evento.keyboard.unichar <= 'Z')
            {
                strcat(str, temp);
            }
            else if (evento.keyboard.unichar >= 'a' &&
                     evento.keyboard.unichar <= 'z')
            {
                strcat(str, temp);
            }
        }

        if (evento.keyboard.keycode == ALLEGRO_KEY_BACKSPACE && strlen(str) != 0)
        {
            str[strlen(str) - 1] = '\0';
        }
    }
}

void exibir_texto_centralizado()
{
    if (strlen(str) > 0)
    {
        al_draw_text(fontemenor, al_map_rgb(224, 56, 4), LARGURA / 2,
                     (ALTURA - al_get_font_ascent(fonte)) / 2,
                     ALLEGRO_ALIGN_CENTRE, str);
    }
}

//Função e loop do menu principal
int mainmenu(void) {
    al_stop_timer(timer);
    //Variáveis booleanas
    int jogar = 0;
    int draw = 0;
    int selected = 0;
    int info = 0;
    int ranking = 0;
    int noscore = 0;
    int garagem = 0;
    int estadointermediario = 0;
    //Structs de cada jogador lido
    Player playerlido1;

    Player playerlido2;
    playerlido2.pontos = 0;
    int player2foilido = 0;

    Player playerlido3;
    playerlido3.pontos = 0;
    int player3foilido = 0;

    Player playerlido4;
    playerlido4.pontos = 0;
    int player4foilido = 0;

    Player playerlido5;
    playerlido5.pontos = 0;
    int player5foilido = 0;
    scoreboard = fopen("rank.bin", "rb");
    if(scoreboard == NULL) {
        noscore = 1;
    }
    if (!noscore) {
        fread(&playerlido1, sizeof(Player), 1, scoreboard);
        if(!feof(scoreboard)) {
            fread(&playerlido2, sizeof(Player), 1, scoreboard);
            player2foilido = 1;
        }
        if(!feof(scoreboard)) {
            fread(&playerlido3, sizeof(Player), 1, scoreboard);
            player3foilido = 1;
        }
        if(!feof(scoreboard)) {
            fread(&playerlido4, sizeof(Player), 1, scoreboard);
            player4foilido = 1;
        }
        if(!feof(scoreboard)) {
            fread(&playerlido5, sizeof(Player), 1, scoreboard);
            player5foilido = 1;
        }
        fclose(scoreboard);
    }
    one = al_load_sample("one.wav");
    partidamoto = al_load_sample("partida.wav");
    logovoz = al_load_sample("logosom.wav");
    fundomenu = al_load_bitmap("logo.png");
    historia = al_load_bitmap("texto.png");
    fundorank = al_load_bitmap("fundorank.png");
    todasmotos = al_load_bitmap("motosgaragem.png");
    textorank = al_load_bitmap("rankingtexto.png");
    al_play_sample(one, 1.3, 0.0, 1.0, ALLEGRO_PLAYMODE_LOOP, &oneid);
    al_draw_bitmap(fundomenu,0,0,0);
    al_draw_text(fonte, al_map_rgb(222, 41, 13), 180, 250, ALLEGRO_ALIGN_CENTRE, "Jogar");
    al_draw_text(fonte, al_map_rgb(130, 143, 19), 180, 320, ALLEGRO_ALIGN_CENTRE, "Garagem");
    al_draw_text(fonte, al_map_rgb(130, 143, 19), 180, 390, ALLEGRO_ALIGN_CENTRE, "Informacoes");
    al_draw_text(fonte, al_map_rgb(130, 143, 19), 180, 460, ALLEGRO_ALIGN_CENTRE, "Ranking");
    al_draw_text(fonte, al_map_rgb(130, 143, 19), 180, 530, ALLEGRO_ALIGN_CENTRE, "Sair");
    al_flip_display();
    while (!jogar) {
        ALLEGRO_EVENT eventomenu;
        al_wait_for_event(queue, &eventomenu);
        if (estadointermediario) {
            manipular_entrada(eventomenu);
        }
        if (eventomenu.type == ALLEGRO_EVENT_KEY_DOWN) {
            switch (eventomenu.keyboard.keycode){
            case ALLEGRO_KEY_DOWN:
                if (garagem) {
                    if (motoselected < 4) {
                        motoselected++;
                    }
                } else {
                    if (selected < 4) {
                        selected++;
                    }
                }
                break;
            case ALLEGRO_KEY_UP:
                if (garagem) {
                    if (motoselected > 0) {
                        motoselected--;
                    }
                } else {
                    if (selected > 0) {
                        selected--;
                    }
                }
                break;
            case ALLEGRO_KEY_ENTER:
                if (estadointermediario) {
                    estadointermediario = 0;
                    al_stop_sample(&oneid);
                    al_play_sample(logovoz, 1.5, 0.0, 1.0, ALLEGRO_PLAYMODE_ONCE, NULL);
                    al_play_sample(partidamoto, 0.5, 0.0, 1.0, ALLEGRO_PLAYMODE_ONCE, NULL);
                    al_rest(4);
                    al_start_timer(timer);
                    jogar = 1;
                }
                if (!info && !ranking && !garagem && !estadointermediario) {
                    if (selected == 4) {
                        return 0;
                        jogar = 1;
                    } else if (selected == 0) {
                        estadointermediario = 1;
                    } else if (selected == 1) {
                        garagem = 1;
                    } else if (selected == 2) {
                        info = 1;
                    } else if (selected == 3) {
                        ranking = 1;
                    }
                } else if (info) {
                    info = 0;
                } else if (ranking) {
                    ranking = 0;
                } else if (garagem) {
                    garagem = 0;
                }
                break;
            }
            draw = 1;
        } else if (eventomenu.type == ALLEGRO_EVENT_DISPLAY_CLOSE) {
            return 0;
            jogar = 1;
        }

        if(draw && al_is_event_queue_empty(queue)) {
            if (!info && !ranking && !garagem && !estadointermediario) {
                al_draw_bitmap(fundomenu,0,0,0);
                if (selected == 0) {
                    al_draw_text(fonte, al_map_rgb(222, 41, 13), 180, 250, ALLEGRO_ALIGN_CENTRE, "Jogar");
                    al_draw_text(fonte, al_map_rgb(130, 143, 19), 180, 320, ALLEGRO_ALIGN_CENTRE, "Garagem");
                    al_draw_text(fonte, al_map_rgb(130, 143, 19), 180, 390, ALLEGRO_ALIGN_CENTRE, "Informacoes");
                    al_draw_text(fonte, al_map_rgb(130, 143, 19), 180, 460, ALLEGRO_ALIGN_CENTRE, "Ranking");
                    al_draw_text(fonte, al_map_rgb(130, 143, 19), 180, 530, ALLEGRO_ALIGN_CENTRE, "Sair");
                } else if (selected == 1) {
                    al_draw_text(fonte, al_map_rgb(130, 143, 19), 180, 250, ALLEGRO_ALIGN_CENTRE, "Jogar");
                    al_draw_text(fonte, al_map_rgb(222, 41, 13), 180, 320, ALLEGRO_ALIGN_CENTRE, "Garagem");
                    al_draw_text(fonte, al_map_rgb(130, 143, 19), 180, 390, ALLEGRO_ALIGN_CENTRE, "Informacoes");
                    al_draw_text(fonte, al_map_rgb(130, 143, 19), 180, 460, ALLEGRO_ALIGN_CENTRE, "Ranking");
                    al_draw_text(fonte, al_map_rgb(130, 143, 19), 180, 530, ALLEGRO_ALIGN_CENTRE, "Sair");
                } else if (selected == 2) {
                    al_draw_text(fonte, al_map_rgb(130, 143, 19), 180, 250, ALLEGRO_ALIGN_CENTRE, "Jogar");
                    al_draw_text(fonte, al_map_rgb(130, 143, 19), 180, 320, ALLEGRO_ALIGN_CENTRE, "Garagem");
                    al_draw_text(fonte, al_map_rgb(222, 41, 13), 180, 390, ALLEGRO_ALIGN_CENTRE, "Informacoes");
                    al_draw_text(fonte, al_map_rgb(130, 143, 19), 180, 460, ALLEGRO_ALIGN_CENTRE, "Ranking");
                    al_draw_text(fonte, al_map_rgb(130, 143, 19), 180, 530, ALLEGRO_ALIGN_CENTRE, "Sair");
                } else if (selected == 3) {
                    al_draw_text(fonte, al_map_rgb(130, 143, 19), 180, 250, ALLEGRO_ALIGN_CENTRE, "Jogar");
                    al_draw_text(fonte, al_map_rgb(130, 143, 19), 180, 320, ALLEGRO_ALIGN_CENTRE, "Garagem");
                    al_draw_text(fonte, al_map_rgb(130, 143, 19), 180, 390, ALLEGRO_ALIGN_CENTRE, "Informacoes");
                    al_draw_text(fonte, al_map_rgb(222, 41, 13), 180, 460, ALLEGRO_ALIGN_CENTRE, "Ranking");
                    al_draw_text(fonte, al_map_rgb(130, 143, 19), 180, 530, ALLEGRO_ALIGN_CENTRE, "Sair");
                } else if (selected == 4) {
                    al_draw_text(fonte, al_map_rgb(130, 143, 19), 180, 250, ALLEGRO_ALIGN_CENTRE, "Jogar");
                    al_draw_text(fonte, al_map_rgb(130, 143, 19), 180, 320, ALLEGRO_ALIGN_CENTRE, "Garagem");
                    al_draw_text(fonte, al_map_rgb(130, 143, 19), 180, 390, ALLEGRO_ALIGN_CENTRE, "Informacoes");
                    al_draw_text(fonte, al_map_rgb(130, 143, 19), 180, 460, ALLEGRO_ALIGN_CENTRE, "Ranking");
                    al_draw_text(fonte, al_map_rgb(222, 41, 13), 180, 530, ALLEGRO_ALIGN_CENTRE, "Sair");
                }
            } else if (info) {
                al_draw_bitmap(fundomenu,0,0,0);
                al_draw_bitmap(historia,0, 0, 0);
                al_draw_text(fonte, al_map_rgb(222, 41, 13), 180, 530, ALLEGRO_ALIGN_CENTRE, "Voltar");
            } else if (ranking) {
                al_draw_bitmap(fundorank,0,0,0);
                if (!noscore) {
                    al_draw_bitmap(textorank,0,0,0);
                    al_draw_textf(fontemenor, al_map_rgb(224, 56, 4), 165, 160, ALLEGRO_ALIGN_CENTRE, "%s", playerlido1.nome);
                    al_draw_textf(fontemenor, al_map_rgb(224, 56, 4), 200, 190, ALLEGRO_ALIGN_CENTRE, "%d", playerlido1.pontos);
                    if (player2foilido && playerlido2.pontos > 0) {
                        al_draw_textf(fontemenor, al_map_rgb(224, 56, 4), 165, 230, ALLEGRO_ALIGN_CENTRE, "%s", playerlido2.nome);
                        al_draw_textf(fontemenor, al_map_rgb(224, 56, 4), 200, 260, ALLEGRO_ALIGN_CENTRE, "%d", playerlido2.pontos);
                    }
                    if (player3foilido && playerlido3.pontos > 0) {
                        al_draw_textf(fontemenor, al_map_rgb(224, 56, 4), 165, 297, ALLEGRO_ALIGN_CENTRE, "%s", playerlido3.nome);
                        al_draw_textf(fontemenor, al_map_rgb(224, 56, 4), 200, 327, ALLEGRO_ALIGN_CENTRE, "%d", playerlido3.pontos);
                    }
                    if (player4foilido && playerlido4.pontos > 0) {
                        al_draw_textf(fontemenor, al_map_rgb(224, 56, 4), 165, 365, ALLEGRO_ALIGN_CENTRE, "%s", playerlido4.nome);
                        al_draw_textf(fontemenor, al_map_rgb(224, 56, 4), 200, 395, ALLEGRO_ALIGN_CENTRE, "%d", playerlido4.pontos);
                    }
                    if (player5foilido && playerlido5.pontos > 0 && playerlido5.pontos < 1000000) {
                        al_draw_textf(fontemenor, al_map_rgb(224, 56, 4), 165, 433, ALLEGRO_ALIGN_CENTRE, "%s", playerlido5.nome);
                        al_draw_textf(fontemenor, al_map_rgb(224, 56, 4), 200, 463, ALLEGRO_ALIGN_CENTRE, "%d", playerlido5.pontos);
                    }
                    al_draw_text(fonte, al_map_rgb(222, 41, 13), 180, 640, ALLEGRO_ALIGN_CENTRE, "Voltar");
                } else {
                    al_draw_text(fonte, al_map_rgb(224, 56, 4), 180, 250, ALLEGRO_ALIGN_CENTRE, "Sem jogadores");
                    al_draw_text(fonte, al_map_rgb(222, 41, 13), 180, 640, ALLEGRO_ALIGN_CENTRE, "Voltar");
                }
            } else if (garagem) {
                al_draw_bitmap(fundomenu,0,0,0);
                al_draw_bitmap(todasmotos, 80, 600, 0);
                if (motoselected == 0) {
                    al_draw_text(fonte, al_map_rgb(222, 41, 13), 180, 250, ALLEGRO_ALIGN_CENTRE, "Moto Chamas");
                    al_draw_text(fonte, al_map_rgb(130, 143, 19), 180, 320, ALLEGRO_ALIGN_CENTRE, "Moto Laranja");
                    al_draw_text(fonte, al_map_rgb(130, 143, 19), 180, 390, ALLEGRO_ALIGN_CENTRE, "Moto Azul");
                    al_draw_text(fonte, al_map_rgb(130, 143, 19), 180, 460, ALLEGRO_ALIGN_CENTRE, "Moto USA");
                    al_draw_text(fonte, al_map_rgb(130, 143, 19), 180, 530, ALLEGRO_ALIGN_CENTRE, "Moto Vermelha");
                } else if (motoselected == 1) {
                    al_draw_text(fonte, al_map_rgb(130, 143, 19), 180, 250, ALLEGRO_ALIGN_CENTRE, "Moto Chamas");
                    al_draw_text(fonte, al_map_rgb(222, 41, 13), 180, 320, ALLEGRO_ALIGN_CENTRE, "Moto Laranja");
                    al_draw_text(fonte, al_map_rgb(130, 143, 19), 180, 390, ALLEGRO_ALIGN_CENTRE, "Moto Azul");
                    al_draw_text(fonte, al_map_rgb(130, 143, 19), 180, 460, ALLEGRO_ALIGN_CENTRE, "Moto USA");
                    al_draw_text(fonte, al_map_rgb(130, 143, 19), 180, 530, ALLEGRO_ALIGN_CENTRE, "Moto Vermelha");
                } else if (motoselected == 2) {
                    al_draw_text(fonte, al_map_rgb(130, 143, 19), 180, 250, ALLEGRO_ALIGN_CENTRE, "Moto Chamas");
                    al_draw_text(fonte, al_map_rgb(130, 143, 19), 180, 320, ALLEGRO_ALIGN_CENTRE, "Moto Laranja");
                    al_draw_text(fonte, al_map_rgb(222, 41, 13), 180, 390, ALLEGRO_ALIGN_CENTRE, "Moto Azul");
                    al_draw_text(fonte, al_map_rgb(130, 143, 19), 180, 460, ALLEGRO_ALIGN_CENTRE, "Moto USA");
                    al_draw_text(fonte, al_map_rgb(130, 143, 19), 180, 530, ALLEGRO_ALIGN_CENTRE, "Moto Vermelha");
                } else if (motoselected == 3) {
                    al_draw_text(fonte, al_map_rgb(130, 143, 19), 180, 250, ALLEGRO_ALIGN_CENTRE, "Moto Chamas");
                    al_draw_text(fonte, al_map_rgb(130, 143, 19), 180, 320, ALLEGRO_ALIGN_CENTRE, "Moto Laranja");
                    al_draw_text(fonte, al_map_rgb(130, 143, 19), 180, 390, ALLEGRO_ALIGN_CENTRE, "Moto Azul");
                    al_draw_text(fonte, al_map_rgb(222, 41, 13), 180, 460, ALLEGRO_ALIGN_CENTRE, "Moto USA");
                    al_draw_text(fonte, al_map_rgb(130, 143, 19), 180, 530, ALLEGRO_ALIGN_CENTRE, "Moto Vermelha");
                } else if (motoselected == 4) {
                    al_draw_text(fonte, al_map_rgb(130, 143, 19), 180, 250, ALLEGRO_ALIGN_CENTRE, "Moto Chamas");
                    al_draw_text(fonte, al_map_rgb(130, 143, 19), 180, 320, ALLEGRO_ALIGN_CENTRE, "Moto Laranja");
                    al_draw_text(fonte, al_map_rgb(130, 143, 19), 180, 390, ALLEGRO_ALIGN_CENTRE, "Moto Azul");
                    al_draw_text(fonte, al_map_rgb(130, 143, 19), 180, 460, ALLEGRO_ALIGN_CENTRE, "Moto USA");
                    al_draw_text(fonte, al_map_rgb(222, 41, 13), 180, 530, ALLEGRO_ALIGN_CENTRE, "Moto Vermelha");
                }
            } else if (estadointermediario) {
                al_draw_bitmap(fundomenu,0,0,0);
                al_draw_text(fontemenor, al_map_rgb(224, 56, 4), 180, 280, ALLEGRO_ALIGN_CENTRE, "INSIRA SEU NOME:");
                exibir_texto_centralizado();
            }
            al_flip_display();
            draw = 0;
        }
    }
    return 1;
}

//Loop/função para a tela de game over
int morto(score) {
    //Variáveis booleanas
    int falencido = 1;
    int selected = 0;
    int draw = 0;
    int concluido = 1;
    int scorenull = 0;
    //Structs para os 5 melhores colocados e o jogador atual
    Player playeratual;
    Player playerlido1;
    int player1existe = 1;
    Player playerlido2;
    int player2existe = 0;
    Player playerlido3;
    int player3existe = 0;
    Player playerlido4;
    int player4existe = 0;
    Player playerlido5;
    int player5existe = 0;
    playeratual.pontos = score;
    strcpy(playeratual.nome, str);
    int primeiro = 0;
    int segundo = 0;
    int terceiro = 0;
    int quarto = 0;
    int quinto = 0;
    //Abre e verifica a existência do ranking
    scoreboard = fopen("rank.bin", "r+b");
    if(scoreboard == NULL) {
        scoreboard = fopen("rank.bin", "w+b");
        scorenull = 1;
        concluido = 0;
    }
    concluido = 0;
    if (!scorenull) {
        //Verifica a existência e lê os 5 primeiros colocados no ranking
        fread(&playerlido1, sizeof(Player), 1, scoreboard);
        if (!feof(scoreboard)) {
            fread(&playerlido2, sizeof(Player), 1, scoreboard);
            player2existe = 1;
        }
        if (!feof(scoreboard)) {
            fread(&playerlido3, sizeof(Player), 1, scoreboard);
            player3existe = 1;
        }
        if (!feof(scoreboard)) {
            fread(&playerlido4, sizeof(Player), 1, scoreboard);
            player4existe = 1;
        }
        if (!feof(scoreboard)) {
            fread(&playerlido5, sizeof(Player), 1, scoreboard);
            player5existe = 1;
        }

        //Determina a posição do jogador atual no ranking
        if(playeratual.pontos >= playerlido1.pontos) {
            primeiro = 1;
        }
        if (player2existe) {
            if (playeratual.pontos >= playerlido2.pontos && playeratual.pontos < playerlido1.pontos) {
                segundo = 1;
            }
        } else {
            segundo = 1;
        }

        if (segundo && primeiro) {
            segundo = 0;
        }

        if (player3existe) {
            if(playeratual.pontos >= playerlido3.pontos && playeratual.pontos < playerlido2.pontos) {
                terceiro = 1;
            }
        } else {
                terceiro = 1;
        }

        if ((terceiro && segundo) || (terceiro && primeiro)) {
            terceiro = 0;
        }

        if (player4existe) {
            if(playeratual.pontos >= playerlido4.pontos && playeratual.pontos < playerlido3.pontos) {
                quarto = 1;
            }
        } else {
            quarto = 1;
        }

        if ((quarto && terceiro) || (quarto && segundo) || (quarto && primeiro)) {
            quarto = 0;
        }

        if (player5existe) {
            if(playeratual.pontos >= playerlido5.pontos && playeratual.pontos < playerlido4.pontos) {
                quinto = 1;
            }
        } else {
            if(player4existe) {
                quinto = 1;
            }
        }

        if ((quinto && terceiro) || (quinto && segundo) || (quinto && quarto) || (quinto && primeiro)) {
            quinto = 0;
        }
    } else {
        primeiro = 1;
        player1existe = 0;
    }

    //Escreve o jogador no ranking
    if (primeiro) {
        fseek(scoreboard, 0, SEEK_SET);
        fwrite(&playeratual, sizeof(Player), 1, scoreboard);
        if (player1existe) {
            fwrite(&playerlido1, sizeof(Player), 1, scoreboard);
        }
        if (player2existe) {
            fwrite(&playerlido2, sizeof(Player), 1, scoreboard);
        }
        if (player3existe) {
            fwrite(&playerlido3, sizeof(Player), 1, scoreboard);
        }
        if (player4existe) {
            fwrite(&playerlido4, sizeof(Player), 1, scoreboard);
        }
        if (player5existe) {
            fwrite(&playerlido5, sizeof(Player), 1, scoreboard);
        }
    }
    if (segundo) {
        fseek(scoreboard, sizeof(Player), SEEK_SET);
        fwrite(&playeratual, sizeof(Player), 1, scoreboard);
        if (player2existe) {
            fwrite(&playerlido2, sizeof(Player), 1, scoreboard);
        }
        if (player3existe) {
            fwrite(&playerlido3, sizeof(Player), 1, scoreboard);
        }
        if (player4existe) {
            fwrite(&playerlido4, sizeof(Player), 1, scoreboard);
        }
        if (player5existe) {
            fwrite(&playerlido5, sizeof(Player), 1, scoreboard);
        }
    }
    if (terceiro) {
        fseek(scoreboard, 2*sizeof(Player), SEEK_SET);
        fwrite(&playeratual, sizeof(Player), 1, scoreboard);
        if (player3existe) {
            fwrite(&playerlido3, sizeof(Player), 1, scoreboard);
        }
        if (player4existe) {
            fwrite(&playerlido4, sizeof(Player), 1, scoreboard);
        }
        if (player5existe) {
            fwrite(&playerlido5, sizeof(Player), 1, scoreboard);
        }
    }
    if (quarto) {
        fseek(scoreboard, 3*sizeof(Player), SEEK_SET);
        fwrite(&playeratual, sizeof(Player), 1, scoreboard);
        if (player4existe) {
            fwrite(&playerlido4, sizeof(Player), 1, scoreboard);
        }
        if (player5existe) {
            fwrite(&playerlido5, sizeof(Player), 1, scoreboard);
        }
    }
    if (quinto) {
        fseek(scoreboard, 4*sizeof(Player), SEEK_SET);
        fwrite(&playeratual, sizeof(Player), 1, scoreboard);
        if (player5existe) {
            fwrite(&playerlido5, sizeof(Player), 1, scoreboard);
        }
    }
    if (!primeiro && !segundo && !terceiro && !quarto && !quinto) {
        fseek(scoreboard, 0, SEEK_END);
        fwrite(&playeratual, sizeof(Player), 1, scoreboard);
    }
    fclose(scoreboard);
    concluido = 1;
    primeiro = 0;
    segundo = 0;
    terceiro = 0;
    quarto = 0;
    quinto = 0;

    //Load dos assets/Renderização de textos
    gameover = al_load_bitmap("gameover.png");
    al_draw_bitmap(gameover,0,0,0);
    al_draw_text(fonte, al_map_rgb(222, 41, 13), 180, 300, ALLEGRO_ALIGN_CENTRE, "Tentar novamente");
    al_draw_text(fonte, al_map_rgb(130, 143, 19), 180, 370, ALLEGRO_ALIGN_CENTRE, "Menu Principal");
    al_flip_display();
    while (falencido) {
        ALLEGRO_EVENT eventomorto;
        al_wait_for_event(queue, &eventomorto);
        if (eventomorto.type == ALLEGRO_EVENT_KEY_DOWN) {
            switch (eventomorto.keyboard.keycode){
            case ALLEGRO_KEY_DOWN:
                if (selected < 1) {
                    selected++;
                }
                break;
            case ALLEGRO_KEY_UP:
                if (selected > 0) {
                    selected--;
                }
                break;
            case ALLEGRO_KEY_ENTER:
                if (selected == 0) {
                    falencido = 0;
                    return 1;
                } else if (selected == 1) {
                    falencido = 0;
                    return 0;
                }
            }
            draw = 1;
        } else if (eventomorto.type == ALLEGRO_EVENT_DISPLAY_CLOSE) {
            falencido = 0;
            return -1;
        }
        //Atualiza a tela
        if(draw && al_is_event_queue_empty(queue)) {
            al_draw_bitmap(gameover,0,0,0);
            if (selected == 0) {
                al_draw_text(fonte, al_map_rgb(222, 41, 13), 180, 300, ALLEGRO_ALIGN_CENTRE, "Tentar novamente");
                al_draw_text(fonte, al_map_rgb(130, 143, 19), 180, 370, ALLEGRO_ALIGN_CENTRE, "Menu Principal");
            } else if (selected == 1) {
                al_draw_text(fonte, al_map_rgb(130, 143, 19), 180, 300, ALLEGRO_ALIGN_CENTRE, "Tentar novamente");
                al_draw_text(fonte, al_map_rgb(222, 41, 13), 180, 370, ALLEGRO_ALIGN_CENTRE, "Menu Principal");
            }
            al_flip_display();
        }
    }
}

int main(void)
{
    //Variaveis booleanas da funcao principal
    int sair = 0;
    int tecla = 0;
    int desenha = 1;
    int dano = 0;
    int pausado = 0;
    int mortoresultado;
    int menuresultado;
    //Invoca a função inicialização e caso ela retorne 0 (erro) é encerrado o programa.
    if (!inicializacao()){
        return -1;
    }

    //Invoca a função mainmenu, caso ela retorne 0 o loop principal não é iniciado e o jogo é encerrado.
    if (!mainmenu()) {
        sair = 1;
    }

    //Variáveis do personagem
    int personagemx = 160;
    int personagemy = 500;
    int hp = 1;
    int pontuacao = 0;

    //Variáveis do carro vermelho
    int carrovermx = 30;
    int carrovermy = 70;
    int carrovermvel = 4;

    //Variáveis do carro azul
    int carroazulx = 260;
    int carroazuly = 100;
    int carroazulvel = 3;

    //Loada as imagens/música
    background = al_load_bitmap(BACKGROUND_ARQUIVO);
    switch (motoselected) {
    case 0:
        personagem = al_load_bitmap("moto1.png");
        break;
    case 1:
        personagem = al_load_bitmap("moto2.png");
        break;
    case 2:
        personagem = al_load_bitmap("moto3.png");
        break;
    case 3:
        personagem = al_load_bitmap("moto4.png");
        break;
    case 4:
        personagem = al_load_bitmap("moto5.png");
        break;
    }
    carroazul = al_load_bitmap(CARROAZUL);
    carroverm = al_load_bitmap(CARROVERM);
    musicamain = al_load_sample("musicamain.wav");
    crashsom = al_load_sample("crash.wav");
    //Renderiza todos assets
    al_play_sample(musicamain, 1.0, 0.0, 1.0, ALLEGRO_PLAYMODE_LOOP, NULL);
    al_draw_bitmap(background,0,0,0);
    al_draw_bitmap(personagem, personagemx, personagemy, 0);
    al_draw_bitmap(carroazul, carroazulx, carroazuly, 0);
    al_draw_bitmap(carroverm, carrovermx, carrovermy, 0);
    al_flip_display();
    //Loop principal
    while(!sair) {
        //Listener de eventos do loop principal
        ALLEGRO_EVENT evento;
        al_wait_for_event(queue, &evento);
        //Detecta o pressionar de uma tecla e trata o caso de cada tecla pressionada
        if(evento.type == ALLEGRO_EVENT_KEY_DOWN) {
            switch(evento.keyboard.keycode) {
            case ALLEGRO_KEY_LEFT:
                tecla = 1;
                break;
            case ALLEGRO_KEY_RIGHT:
                tecla = 2;
                break;
            case ALLEGRO_KEY_ESCAPE:
                sair = 1;
            case ALLEGRO_KEY_SPACE:
                if (al_get_timer_started(timer)) {
                    al_stop_timer(timer);
                } else {
                    al_start_timer(timer);
                }
            }
        } else if (evento.type == ALLEGRO_EVENT_KEY_UP) {
            tecla = 0;
        } else if (evento.type == ALLEGRO_EVENT_DISPLAY_CLOSE) {
            sair = 1;
        }
        //Ocorre a cada evento do timer/fps
        if(evento.type == ALLEGRO_EVENT_TIMER) {
            if (tecla == 1) {
                //Impede personagem de sair da tela pela esquerda e o movimenta para a esquerda
                if (personagemx > 0) {
                    personagemx -= 3;
                }
            } else if (tecla == 2) {
                //Impede personagem de sair da tela pela direita e o movimenta para a direita
                if (personagemx < 320) {
                    personagemx += 3;
                }
            }

            //Reseta a posição dos carros e incrementa a pontuação a cada ultrapassagem
            if (carroazuly > 725) {
                carroazuly = -100;
                carroazulx = random(190, 280);
                pontuacao++;
            } else {
                carroazuly += carroazulvel;
            }

            if (carrovermy > 725) {
                carrovermy = -100;
                carrovermx = random(25, 100);
                pontuacao++;
            } else {
                carrovermy += carrovermvel;
            }

            //Colisoes carro azul (hitbox)
            if ((carroazulx <= (personagemx+30)) && ((carroazulx+60) >= personagemx)) {
                if (((carroazuly + 132) >= personagemy) && (carroazuly <= (personagemy+80))) {
                    dano = 1;
                }
            }

            //Colisoes carro vermelho (hitbox)
            if ((carrovermx <= (personagemx+30)) && ((carrovermx+60) >= personagemx)) {
                if (((carrovermy + 132) >= personagemy) && (carrovermy <= (personagemy+80))) {
                    dano = 1;
                }
            }

            //Aumenta a dificultade do jogo (velocidade dos carros) a cada intervalo de pontuação
            if (pontuacao < 5) {
                carroazulvel = 3;
                carrovermvel = 4;
            } else if (pontuacao >= 5 && pontuacao < 10) {
                carroazulvel = 4;
                carrovermvel = 5;
            } else if (pontuacao >= 10 && pontuacao < 20) {
                carroazulvel = 5;
                carrovermvel = 6;
            } else if (pontuacao >= 20 && pontuacao < 30) {
                carroazulvel = 6;
                carrovermvel = 7;
            } else if (pontuacao >= 30 && pontuacao < 40) {
                carroazulvel = 7;
                carrovermvel = 8;
            } else if (pontuacao >= 40 && pontuacao < 50) {
                carroazulvel = 8;
                carrovermvel = 9;
            } else if (pontuacao >= 50 && pontuacao < 60) {
                carroazulvel = 9;
                carrovermvel = 10;
            } else if (pontuacao >= 60 && pontuacao < 70) {
                carroazulvel = 10;
                carrovermvel = 11;
            } else if (pontuacao >= 70 && pontuacao < 80) {
                carroazulvel = 11;
                carrovermvel = 12;
            }

            //Trata a morte do personagem/game over
            if (hp <= 0) {
                //Para a música
                al_stop_samples();
                //Invoca a função/loop da tela de gameover e atribui seu resultado a uma variável por possuir mais de 2
                al_play_sample(crashsom, 1.0, 0.0, 1.0, ALLEGRO_PLAYMODE_ONCE, NULL);
                mortoresultado = morto(pontuacao);
                //Caso Tentar Novamente (jogo reinicia)
                if(mortoresultado == 1) {
                    al_play_sample(musicamain, 1.0, 0.0, 1.0, ALLEGRO_PLAYMODE_LOOP, NULL);
                    hp = 5;
                    pontuacao = 0;
                    carrovermy = -100;
                    carroazuly = -100;
                } else if (mortoresultado == 0) {
                    //Caso Voltar ao menu principal
                    menuresultado = mainmenu();
                    //Caso sair seja selecionado no menu principal
                    if(menuresultado == 0) {
                        sair = 1;
                    }
                    switch (motoselected) {
                    case 0:
                        personagem = al_load_bitmap("moto1.png");
                        break;
                    case 1:
                        personagem = al_load_bitmap("moto2.png");
                        break;
                    case 2:
                        personagem = al_load_bitmap("moto3.png");
                        break;
                    case 3:
                        personagem = al_load_bitmap("moto4.png");
                        break;
                    case 4:
                        personagem = al_load_bitmap("moto5.png");
                        break;
                    }
                    al_play_sample(musicamain, 1.0, 0.0, 1.0, ALLEGRO_PLAYMODE_LOOP, NULL);
                    hp = 5;
                    pontuacao = 0;
                    carrovermy = -100;
                    carroazuly = -100;
                } else if (mortoresultado == -1) {
                    //Caso a tela seja fechada
                    sair = 1;
                }
            }
            desenha = 1;
        }

        //Atualiza a tela
        if(desenha && al_is_event_queue_empty(queue)) {
            al_draw_bitmap(background,0,0,0);
            al_draw_bitmap(personagem,personagemx, personagemy, 0);
            al_draw_bitmap(carroazul, carroazulx, carroazuly, 0);
            al_draw_bitmap(carroverm, carrovermx, carrovermy, 0);
            if(dano) {
                hp -= 1;
            }
            al_draw_textf(fonte, al_map_rgb(33, 103, 217), 230, 0, ALLEGRO_ALIGN_CENTRE, "Pontuacao: %d", pontuacao);
            al_flip_display();
            dano = 0;
            desenha = 0;
        }
    }

    //Encerra o programa
    al_destroy_timer(timer);
    al_destroy_display(display);
    al_destroy_event_queue(queue);

    return 0;
}
