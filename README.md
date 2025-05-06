# Noyau_temps_reel_FreeRTOS

##01. Premier pas

1) le fichier main.c se sittue ici : Core\Src\main.c

2) Ce sont les zones qui vont être protégé lors de chaque régénération du code, il est donc fortement conseillé d'écrire entre le BEGIN et le END sous peine de perdre sont code si l'on regénère le projet.

3) Dans la fonction HAL_GPIO_TogglePin on doit passer en paramètre Le type de GPIO qu'on veut utiliser dans notre cas "GPIOI" ainsi que la PIN qu'on utilise pour nous c'est "GPIO_PIN_1". Et dans le HAL_Delay, on passe en paramètre le temps en ms qu'on veut entre chaque itération du while, nous avons choisit 500ms dans notre code.

4) Les entrées/sorties sont définie dans le fichier Core\Src\gpio.c .

5) Pour faire clignoter la LED on écrit ce programme dans le while (1)
   HAL_GPIO_TogglePin(GPIOI, GPIO_PIN_1);
   HAL_Delay(500);

6) Pour ajouter la fonctionnalité d'allumage de la LED avec l'appuie du boutton, on modifie le code dans le while(1) par ça :
  if (HAL_GPIO_ReadPin(GPIOI, GPIO_PIN_11) == GPIO_PIN_SET)
		{
		  HAL_GPIO_WritePin(GPIOI, GPIO_PIN_1, GPIO_PIN_RESET);
		}
  else
	  {
		  HAL_GPIO_WritePin(GPIOI, GPIO_PIN_1, GPIO_PIN_SET);
	  }
  HAL_Delay(50);

 ## 1. FreeRTOS, Tâches et semaphores

 ### 1.1 tache simple

 1) TOTAL_HEAP_SIZE défini l'espace que l'on accorde au Kernel de FreeRTOS sur la RAM. Lorsque on change ce paramètre, un define vois sa valeur changer dans FreeRTOSConfig.h.

 2) Creation d'une tache simple : xTaskCreate(task1, "Task1", 128, NULL, 1, NULL);
TICK_PERIOD_MS permet de définir le temps d'attente entre chaque exécution de la fonction.

### 1.3 Notification

Exemple utilisation notification : xTaskNotifyGive(taskHandle); // Notifie une tache
				   ulTaskNotifyTake(pdTRUE, portMAX_DELAY); // Attend une notification de tâche

### 1.5 Réentrance et exclusion mutuelle

10) Printf est une ressource partagée, or celle ci est utilisée par nos deux tâches. Les deux taches essayent d’écrire en même temps sur l’affichage, ainsi, les messages sont incorrect car subissent des interférences
    
11) Utiliser un mutex pour garantir que seul une tâche à la fois accéde au printf :`

    /*-----Création du mutex : ---------*/

    xMutex = xSemaphoreCreateMutex();
	if (xMutex == NULL) {
 		printf("Erreur : Mutex non créé\r\n");
    		while (1);
	}
 
/* --------- Utilisation du mutex : --------*/
void task_bug(void * pvParameters)
{
    int delay = (int) pvParameters;
    for(;;)
    {
        if (xSemaphoreTake(xMutex, portMAX_DELAY) == pdTRUE) {  // Prend le mutex
            printf("Je suis %s et je m'endors pour %d ticks\r\n", pcTaskGetName(NULL), delay);
            xSemaphoreGive(xMutex);
        }
        vTaskDelay(delay);
    }
}

## 2 On joue avec le Shell

1.3 Lorsqu'on tape une commande dans le Shell, "shell_run()" lit les caractères depuis "drv_uart1_receive". Le shell cherche la commande assimilée à la commande ('f' par exemple dans le cas de notre "fonction inutile") dans la liste "shell_add()". Pour terminer, il appelle la fonction correspondante ("fonction" dans notre cas) avec ses arguments.

1.4 La fonction est exécutée dans le même thread que le shell ainsi, si une fonction bloque, le shell ne répond plus

1.5 On peut resoudre ce probleme en mettant les traitement long à une tache FREERTOS dédié et en utilisant des Queues ou variables partagées pour les communications

2) Le bon respect des priorités est indispensable, ces priorités permettent de décision quelle tâche est exécutée anvant l’autre.
   
## 3 Debug, gestion d’erreur et statistiques

### 3.1 Gestion  du tas

1) Il s'agit de la "heap", en FreeRTOS, cette mémoire est gérée par le mécanisme dans la heap,  sa taille est définie via "configTOTAL_HEAP_SIZE" directement dans FreeRTOSConfig.h
2) C'est géré par FreeRTOS
4) Memoire FLASH : 29.27KB soit 2.86% ; Memoire RAM : 18.79KB soit 5.87%
5) Les taches sont créées jusqu’à l’arrivée d'une erreur
7) La taille du tas a été modifiée, en revanche nous ne sommes pas parvenus à afficher la nouvelle mémoire utilisée

### 3.2 Gestion des piles

4) On utilise une LED sur la carte qui s'allume lorsque la pile est remplie

Cette partie de programme nous permet, via la commande 'o' de créer volontairement une tâche avec une stack trop petite (64 mots), pour provoquer un dépassement de pile (stack overflow)

int stackOverflowShellFunction(h_shell_t * h_shell, int argc, char ** argv)
{
    int len = snprintf(h_shell->print_buffer, BUFFER_SIZE,
    "Triggering stack overflow...\r\n");
    h_shell->drv.transmit(h_shell->print_buffer, len);
    if (xTaskCreate(dummyTask, "OverflowTask", 64, NULL, 2, NULL) != pdPASS) {
        int len = snprintf(h_shell->print_buffer, BUFFER_SIZE, "Error creating OverflowTask\r\n");
        h_shell->drv.transmit(h_shell->print_buffer, len);
    }
    return 0;
}

void vApplicationStackOverflowHook(TaskHandle_t xTask, char *pcTaskName)
{
    //allume une LED si stack overflow détecté
    HAL_GPIO_WritePin(GPIOI, GPIO_PIN_1, GPIO_PIN_SET);
    while (1);
}


### 3.3 Statistiques dans l’IDE

--> en cours


