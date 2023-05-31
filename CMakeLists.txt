cmake_minimum_required(VERSION 3.15.3)

project(nucleo-F446RE)

enable_language(CXX C ASM)
set(CMAKE_C_STANDARD 17)
set(CMAKE_C_STANDARD_REQUIRED True)
set(CMAKE_C_EXTENSIONS False)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_C_STANDARD_REQUIRED True)
set(CMAKE_C_EXTENSIONS False)

set(STM32CUBEMX_GENERATED_FILES

		library/Middlewares/Third_Party/FreeRTOS/Source/croutine.c
		library/Middlewares/Third_Party/FreeRTOS/Source/event_groups.c
		library/Middlewares/Third_Party/FreeRTOS/Source/list.c
		library/Middlewares/Third_Party/FreeRTOS/Source/queue.c
		library/Middlewares/Third_Party/FreeRTOS/Source/stream_buffer.c
		library/Middlewares/Third_Party/FreeRTOS/Source/tasks.c
		library/Middlewares/Third_Party/FreeRTOS/Source/timers.c

		library/Middlewares/Third_Party/FreeRTOS/Source/portable/GCC/ARM_CM4F/port.c

		library/Middlewares/Third_Party/FreeRTOS/Source/portable/MemMang/heap_4.c

		library/Middlewares/Third_Party/FreeRTOS/Source/CMSIS_RTOS_V2/cmsis_os2.c

		library/Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_hal_adc.c
		library/Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_hal_spi.c
		library/Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_hal_tim.c
		library/Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_hal_tim_ex.c
		library/Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_hal_uart.c
		library/Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_hal_i2c.c
		library/Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_hal_rcc.c
		library/Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_hal_rcc_ex.c
		library/Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_hal_flash.c
		library/Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_hal_flash_ex.c
		library/Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_hal_flash_ramfunc.c
		library/Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_hal_gpio.c
		library/Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_hal_dma_ex.c
		library/Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_hal_dma.c
		library/Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_hal_pwr.c
		library/Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_hal_pwr_ex.c
		library/Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_hal_cortex.c
		library/Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_hal.c
		library/Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_hal_exti.c
		library/Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_hal_pcd.c
		library/Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_hal_pcd_ex.c
		library/Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_ll_usb.c

		library/Inc/main.h
		library/Inc/gpio.h
		library/Inc/usart.h
		library/Inc/stm32f4xx_hal_conf.h
		library/Inc/stm32f4xx_it.h
		library/Inc/FreeRTOSConfig.h

		library/Src/main.c
		library/Src/gpio.c
		library/Src/usart.c
		library/Src/stm32f4xx_hal_msp.c
		library/Src/stm32f4xx_it.c
		library/Src/system_stm32f4xx.c
		library/Src/freertos.c
		library/Src/stm32f4xx_hal_timebase_tim.c

		library/startup_stm32f446xx.s)

set(EXECUTABLE ${PROJECT_NAME}.out)

add_executable(${EXECUTABLE} ${STM32CUBEMX_GENERATED_FILES}
)

target_compile_definitions(${EXECUTABLE} PRIVATE
		-DUSE_HAL_DRIVER
		-DSTM32F446xx
)

target_include_directories(${EXECUTABLE} PRIVATE
		library/Inc
		library/Drivers/STM32F4xx_HAL_Driver/Inc
		library/Drivers/CMSIS/Device/ST/STM32F4xx/Include
		library/Drivers/CMSIS/Include

		library/Middlewares/Third_Party/FreeRTOS/Source/include
		library/Middlewares/Third_Party/FreeRTOS/Source/CMSIS_RTOS_V2
		library/Middlewares/Third_Party/FreeRTOS/Source/portable/GCC/ARM_CM4F
)

target_compile_options(${EXECUTABLE} PRIVATE
		-mcpu=cortex-m4
		-mthumb
		-mfpu=fpv4-sp-d16
		-mfloat-abi=hard

		-ffunction-sections
		-fdata-sections
		-fno-exceptions
		-Wall
		$<$<CONFIG:Debug>:-O0>
)

target_link_options(${EXECUTABLE} PRIVATE
		-T${CMAKE_SOURCE_DIR}/library/STM32F446RETx_FLASH.ld
		-mcpu=cortex-m4
		-mthumb
		-mfpu=fpv4-sp-d16
		-mfloat-abi=hard
		-lc
		-lm
		-lnosys
		-specs=nosys.specs
		-u _printf_float

		-Wl,-Map=${PROJECT_NAME}.map,--cref
		-Wl,--gc-sections
		-Wl,--print-memory-usage
)

# Print executable size
add_custom_command(TARGET ${EXECUTABLE}
		POST_BUILD
		COMMAND arm-none-eabi-size ${EXECUTABLE})

# Create hex file
add_custom_command(TARGET ${EXECUTABLE}
		POST_BUILD
		COMMAND arm-none-eabi-objcopy -O ihex ${EXECUTABLE} ${PROJECT_NAME}.hex
		COMMAND arm-none-eabi-objcopy -O binary ${EXECUTABLE} ${PROJECT_NAME}.bin)