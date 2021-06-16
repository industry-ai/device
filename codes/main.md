# main loop of the device


```c
  while (1)
  {
    /* USER CODE END WHILE */

    /* USER CODE BEGIN 3 */
	  if ( ACTION_NEW == pAction->yFlag )
	  {
		  ( *pAction->pFuc )( pAction->yID );
		  pAction->yFlag	= ACTION_DONE;

		  pAction	= pAction->pNext;
		  continue;
	  }

	  HAL_StatusTypeDef	HAL_RetVal;

	  if ( SYS_STATE_RESET == sysState )
	  {
		  HAL_GPIO_TogglePin ( LED1_GPIO_Port, LED1_Pin );

//		  continue;
	  }

	  if ( SYS_STATE_PING == sysState )
	  {
		  pData [ 0 ]	= 0x00;
		  pData [ 1 ]	= 0xFE;		// ping-pong code
		  HAL_RetVal	= CANx_SendNormalData ( &hcan, BOARD_ID, pData, 8 );
		  if ( HAL_RetVal == HAL_OK )
		  {
			  HAL_GPIO_TogglePin ( LED1_GPIO_Port, LED1_Pin );

			  sysState	= SYS_STATE_PONG;
		  }
	  }
  }
```


