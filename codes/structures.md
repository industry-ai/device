# main structures and functions


The data structure associated with exective actions is defined in 
`canSTM32F103ZETx\Inc\action.h`.

Structure of actions:
```c
typedef struct tagAction	ACTION, *LPACTION;
typedef void ( *LPFUNC_ACTION )( uint8_t yID );

struct tagAction
{
	LPFUNC_ACTION	pFuc;

	LPACTION	pNext;

	uint8_t		yID;
	uint8_t		yFlag;
};
```
