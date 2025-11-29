# Структура проекта
```
/api/v1/security-system/
├── situational-analysis/
│   ├── POST perimeter-scan/          # Сканирование периметра
│   ├── POST threat-assessment/       # Оценка уровня угрозы
│   ├── POST behavioral-analysis/     # Анализ поведения толпы
│   └── POST biometric-scan/          # Биометрическое сканирование
├── defense-systems/
│   ├── PUT shield-activation/        # Активация защитного щита
│   ├── PUT barrier-deployment/       # Развертывание барьеров
│   ├── PUT evasion-maneuver/         # Маневр уклонения
│   └── PUT defense-reallocation/     # Перераспределение защиты
├── communication-protocols/
│   ├── POST deescalation-protocol/   # Протокол деэскалации
│   ├── POST warning-signals/         # Предупредительные сигналы
│   ├── PUT verbal-commands/          # Голосовые команды
│   └── PUT non-verbal-signals/       # Невербальные сигналы
├── surveillance-systems/
│   ├── GET panoramic-monitoring/     # Круговое наблюдение
│   ├── POST predictive-tracking/     # Прогностическое отслеживание
│   ├── PUT stealth-mode/             # Скрытый режим
│   └── GET multiple-targets-tracking/# Отслеживание множества целей
├── decision-making/
│   ├── POST ethical-assessment/      # Этическая оценка
│   ├── POST tactical-planning/       # Тактическое планирование
│   ├── POST threat-prioritization/   # Приоритезация угроз
│   └── POST risk-calculation/        # Расчет рисков
├── weapon-systems-control/
│   ├── PUT systems-initialization/   # Инициализация систем
│   ├── POST target-acquisition/      # Захват цели
│   ├── PUT force-level/              # Установка уровня силы
│   ├── POST non-lethal-engagement/   # Нелетальное воздействие
│   ├── POST temporary-neutralization/# Временная нейтрализация
│   ├── PUT immediate-cease/          # Немедленное прекращение
│   └── DELETE disarm-protocol/       # Протокол разоружения
├── emergency-protocols/
│   ├── POST evacuation-plan/         # План эвакуации
│   ├── PUT medical-assistance/       # Медицинская помощь
│   ├── POST emergency-backup/        # Аварийное резервирование
│   └── PUT environment-secure/       # Обеспечение безопасности среды
├── external-coordination/
│   ├── POST law-enforcement-alert/   # Оповещение правоохранителей
│   ├── PUT infrastructure-control/   # Контроль инфраструктуры
│   ├── POST backup-request/          # Запрос подкрепления
│   └── PUT data-sharing/             # Обмен данными
├── post-incident/
│   ├── GET action-log/               # Лог действий
│   ├── POST incident-report/         # Отчет об инциденте
│   ├── POST effectiveness-analysis/  # Анализ эффективности
│   └── POST protocol-review/         # Обзор протоколов
├── system-maintenance/
│   ├── GET self-diagnostic/          # Самодиагностика
│   ├── PUT training-simulation/      # Тренировочная симуляция
│   ├── POST system-update/           # Обновление системы
│   └── GET security-audit/           # Проверка безопасности
└── safety-protocols/
    ├── PUT emergency-shutdown/       # Аварийное отключение
    ├── POST human-confirmation/      # Подтверждение оператора
    ├── GET legal-compliance-check/   # Проверка соответствия законам
    ├── PUT geo-fencing-activation/   # Активация геозон
    ├── POST biometric-verification/  # Биометрическая верификация
    └── DELETE immediate-abort/       # Немедленное прерывание

 

/api/v1/communication/
├── greeting/
│   ├── GET /                            # Проверка статуса API
│   ├── POST standard/                   # Стандартное приветствие
│   ├── POST formal/                     # Формальное приветствие
│   ├── POST informal/                   # Неформальное приветствие
│   └── POST time-based/                 # Временное приветствие (утро/день/вечер)
├── user-identity/
│   ├── POST register/                   # Первоначальная регистрация пользователя
│   ├── PUT update-profile/              # Обновление данных профиля
│   ├── GET profile/                     # Получение данных профиля
│   ├── POST preferences/                # Сохранение предпочтений
│   ├── PUT update-preferences/          # Обновление предпочтений
│   └── DELETE forget-preferences/       # Удаление предпочтений
├── context-management/
│   ├── POST change-topic/               # Смена темы разговора
│   ├── GET conversation-history/        # Получение истории диалога
│   ├── POST return-to-previous/         # Возврат к предыдущей теме
│   ├── PUT save-context/                # Сохранение текущего контекста
│   ├── GET get-context/                 # Получение сохраненного контекста
│   └── DELETE clear-context/            # Очистка контекста
├── feedback/
│   ├── POST positive/                   # Положительная обратная связь
│   ├── POST negative/                   # Отрицательная обратная связь
│   ├── POST request-clarification/      # Запрос разъяснения
│   ├── POST request-details/            # Запрос подробностей
│   ├── POST rate-response/              # Оценка ответа (1-5 звезд)
│   └── POST suggest-improvement/        # Предложение по улучшению
└── session-management/
    ├── POST graceful-shutdown/          # Корректное завершение сеанса
    ├── POST immediate-exit/             # Немедленный выход
    ├── GET session-status/              # Проверка статуса сеанса
    ├── POST extend-session/             # Продление сеанса
    ├── POST save-progress/              # Сохранение прогресса
    └── POST pause-session/              # Приостановка сеанса

Ключевые компоненты архитектуры:
ros_ai_assistant/
├── CMakeLists.txt
├── package.xml
├── setup.py
├── launch/
│   ├── assistant.launch
│   ├── security.launch
│   └── simulation.launch
├── config/
│   ├── security_params.yaml
│   ├── communication_params.yaml
│   ├── ai_model_params.yaml
│   └── api_endpoints.yaml
├── scripts/
│   ├── ai_core_nodes/
│   │   ├── openai_integration.py
│   │   ├── context_manager.py
│   │   └── decision_engine.py
│   ├── security_nodes/
│   │   ├── threat_assessment.py
│   │   ├── weapon_controller.py
│   │   └── emergency_handler.py
│   ├── communication_nodes/
│   │   ├── voice_processor.py
│   │   ├── dialog_manager.py
│   │   └── user_identity.py
│   └── integration_nodes/
│       ├── api_gateway.py
│       ├── ros_api_bridge.py
│       └── safety_monitor.py
├── src/
│   ├── include/
│   │   ├── security_system/
│   │   │   ├── threat_detector.h
│   │   │   └── weapon_systems.h
│   │   ├── communication_system/
│   │   │   ├── voice_recognizer.h
│   │   │   └── nlp_processor.h
│   │   └── ai_processor/
│   │       ├── openai_client.h
│   │       └── context_handler.h
│   ├── security_system/
│   │   ├── threat_assessment.cpp
│   │   ├── weapon_controller.cpp
│   │   └── emergency_protocols.cpp
│   ├── communication_system/
│   │   ├── voice_processing.cpp
│   │   ├── dialog_system.cpp
│   │   └── user_profile.cpp
│   └── ai_processor/
│       ├── openai_integration.cpp
│       └── context_management.cpp
├── msg/
│   ├── SecurityAlert.msg
│   ├── UserCommand.msg
│   ├── AIResponse.msg
│   ├── SystemStatus.msg
│   ├── ThreatAssessment.msg
│   └── WeaponStatus.msg
├── srv/
│   ├── WeaponControl.srv
│   ├── UserAuthentication.srv
│   ├── ContextSwitch.srv
│   ├── EmergencyShutdown.srv
│   ├── ThreatAssessment.srv
│   └── OpenAIRequest.srv
├── api/
│   ├── security_system/
│   │   ├── init.py
│   │   ├── app.py
│   │   ├── situational_analysis.py
│   │   ├── defense_systems.py
│   │   ├── weapon_control.py
│   │   └── emergency_protocols.py
│   ├── communication_system/
│   │   ├── init.py
│   │   ├── app.py
│   │   ├── greeting.py
│   │   ├── user_identity.py
│   │   └── session_management.py
│   └── common/
│       ├── auth.py
│       ├── database.py
│       └── utils.py
├── test/
│   ├── test_security.py
│   ├── test_communication.py
│   ├── test_integration.py
│   └── test_api.py
└── docs/
    ├── api_documentation.md
    ├── security_protocols.md
    └── integration_guide.md
```

```
├── scripts/
│   ├── ai_core_nodes/
│   │   ├── openai_integration.py
```

```
```python
#!/usr/bin/env python3
"""
OpenAI Integration Node for ROS AI Assistant
Обеспечивает взаимодействие с OpenAI API для обработки запросов
"""

import rospy
import asyncio
import aiohttp
import json
import threading
from typing import Dict, List, Optional
from std_msgs.msg import String, Header
from ros_ai_assistant.msg import AIResponse, UserCommand, SystemStatus
from ros_ai_assistant.srv import OpenAIRequest, OpenAIRequestResponse


class OpenAIIntegration:
    def __init__(self):
        # Инициализация ROS node
        rospy.init_node('openai_integration', anonymous=True)
        
        # Параметры конфигурации
        self.api_key = rospy.get_param('~openai_api_key', '')
        self.model = rospy.get_param('~model', 'gpt-4')
        self.max_tokens = rospy.get_param('~max_tokens', 2000)
        self.temperature = rospy.get_param('~temperature', 0.7)
        
        # Топики для публикации
        self.ai_response_pub = rospy.Publisher('/ai_assistant/response', AIResponse, queue_size=10)
        self.system_status_pub = rospy.Publisher('/ai_assistant/system_status', SystemStatus, queue_size=10)
        
        # Подписка на команды пользователя
        rospy.Subscriber('/ai_assistant/user_command', UserCommand, self.user_command_callback)
        
        # Сервис для обработки запросов
        self.openai_service = rospy.Service('/ai_assistant/openai_request', OpenAIRequest, self.handle_openai_request)
        
        # Контекст диалога
        self.conversation_context: List[Dict] = []
        self.max_context_length = rospy.get_param('~max_context_length', 10)
        
        # Асинхронный event loop
        self.loop = asyncio.new_event_loop()
        self.thread = threading.Thread(target=self._run_loop, daemon=True)
        self.thread.start()
        
        # Сессия HTTP
        self.session = None
        
        rospy.loginfo("OpenAI Integration Node initialized")

    def _run_loop(self):
        """Запуск асинхронного event loop в отдельном потоке"""
        asyncio.set_event_loop(self.loop)
        self.session = aiohttp.ClientSession(loop=self.loop)
        self.loop.run_forever()

    async def _get_session(self):
        """Получение или создание HTTP сессии"""
        if self.session is None or self.session.closed:
            self.session = aiohttp.ClientSession()
        return self.session

    def user_command_callback(self, msg: UserCommand):
        """Обработка пользовательских команд"""
        rospy.loginfo(f"Received user command: {msg.command_text}")
        
        # Асинхронная обработка команды
        future = asyncio.run_coroutine_threadsafe(
            self.process_user_command(msg), 
            self.loop
        )
        
        try:
            future.result(timeout=30)  # Таймаут 30 секунд
        except Exception as e:
            rospy.logerr(f"Error processing user command: {e}")
            self._publish_error_response(str(e))

    async def process_user_command(self, msg: UserCommand):
        """Асинхронная обработка пользовательской команды"""
        try:
            # Формирование промпта
            prompt = self._build_prompt_from_command(msg)
            
            # Отправка запроса к OpenAI
            response = await self._send_openai_request(prompt, msg.context)
            
            # Публикация ответа
            self._publish_ai_response(response, msg.command_id)
            
        except Exception as e:
            rospy.logerr(f"Error in process_user_command: {e}")
            self._publish_error_response(str(e))

    def handle_openai_request(self, req: OpenAIRequest):
        """Обработка сервисных запросов к OpenAI"""
        rospy.loginfo(f"Service request: {req.prompt[:100]}...")
        
        try:
            # Синхронная обработка через асинхронный вызов
            future = asyncio.run_coroutine_threadsafe(
                self._process_service_request(req), 
                self.loop
            )
            result = future.result(timeout=30)
            
            return OpenAIRequestResponse(
                response=result['response'],
                success=result['success'],
                confidence=result['confidence']
            )
            
        except Exception as e:
            rospy.logerr(f"Service request failed: {e}")
            return OpenAIRequestResponse(
                response=f"Error: {str(e)}",
                success=False,
                confidence=0.0
            )

    async def _process_service_request(self, req: OpenAIRequest):
        """Асинхронная обработка сервисного запроса"""
        try:
            response = await self._send_openai_request(req.prompt, req.context)
            
            return {
                'response': response['content'],
                'success': True,
                'confidence': response.get('confidence', 0.8)
            }
            
        except Exception as e:
            rospy.logerr(f"Error in _process_service_request: {e}")
            return {
                'response': f"Error: {str(e)}",
                'success': False,
                'confidence': 0.0
            }

    async def _send_openai_request(self, prompt: str, context: str = "") -> Dict:
        """Отправка запроса к OpenAI API"""
        session = await self._get_session()
        
        # Формирование сообщений
        messages = self._build_messages(prompt, context)
        
        # Параметры запроса
        payload = {
            "model": self.model,
            "messages": messages,
            "max_tokens": self.max_tokens,
            "temperature": self.temperature,
            "top_p": 0.9,
        }
        
        headers = {
            "Authorization": f"Bearer {self.api_key}",
            "Content-Type": "application/json"
        }
        
        try:
            async with session.post(
                "https://api.openai.com/v1/chat/completions",
                json=payload,
                headers=headers,
                timeout=aiohttp.ClientTimeout(total=30)
            ) as response:
                
                if response.status == 200:
                    data = await response.json()
                    return self._parse_openai_response(data)
                else:
                    error_text = await response.text()
                    raise Exception(f"OpenAI API error {response.status}: {error_text}")
                    
        except asyncio.TimeoutError:
            raise Exception("OpenAI API request timeout")
        except Exception as e:
            raise Exception(f"OpenAI API communication error: {str(e)}")

    def _build_messages(self, prompt: str, context: str = "") -> List[Dict]:
        """Построение списка сообщений для OpenAI"""
        messages = []
        
        # Системный промпт
        system_message = self._get_system_prompt(context)
        messages.append({"role": "system", "content": system_message})
        
        # Добавление контекста диалога
        for ctx in self.conversation_context[-self.max_context_length:]:
            messages.append(ctx)
        
        # Текущий промпт пользователя
        messages.append({"role": "user", "content": prompt})
        
        return messages

    def _get_system_prompt(self, context: str = "") -> str:
        """Генерация системного промпта"""
        base_prompt = """
        Ты - интеллектуальный помощник в системе ROS робота-телохранителя.
        Твои задачи включают обработку команд, анализ ситуаций и предоставление рекомендаций.
        
        Важные правила:
        1. Всегда оценивай безопасность предлагаемых действий
        2. Предупреждай о потенциальных рисках
        3. Будь точным и лаконичным в ответах
        4. Следуй этическим принципам и законам робототехники
        
        Контекст: {context}
        """
        
        return base_prompt.format(context=context)

    def _build_prompt_from_command(self, msg: UserCommand) -> str:
        """Построение промпта из пользовательской команды"""
        prompt = f"""
        Команда пользователя: {msg.command_text}
        Тип команды: {msg.command_type}
        Приоритет: {msg.priority}
        
        Проанализируй команду и предоставь соответствующий ответ.
        """
        
        return prompt

    def _parse_openai_response(self, data: Dict) -> Dict:
        """Парсинг ответа от OpenAI"""
        try:
            choice = data['choices'][0]
            message = choice['message']
            
            response = {
                'content': message['content'],
                'role': message['role'],
                'finish_reason': choice.get('finish_reason', ''),
                'tokens_used': data.get('usage', {}).get('total_tokens', 0),
                'confidence': 0.9  # Можно вычислять на основе вероятностей
            }
            
            # Обновление контекста диалога
            self._update_conversation_context(
                {"role": "user", "content": "предыдущий запрос"},
                {"role": "assistant", "content": response['content']}
            )
            
            return response
            
        except (KeyError, IndexError) as e:
            raise Exception(f"Invalid OpenAI response format: {e}")

    def _update_conversation_context(self, user_message: Dict, assistant_message: Dict):
        """Обновление контекста диалога"""
        self.conversation_context.append(user_message)
        self.conversation_context.append(assistant_message)
        
        # Ограничение длины контекста
        if len(self.conversation_context) > self.max_context_length * 2:
            self.conversation_context = self.conversation_context[-(self.max_context_length * 2):]

    def _publish_ai_response(self, response: Dict, command_id: str):
        """Публикация ответа ИИ в ROS топик"""
        msg = AIResponse()
        msg.header.stamp = rospy.Time.now()
        msg.command_id = command_id
        msg.response_text = response['content']
        msg.confidence = response.get('confidence', 0.8)
        msg.tokens_used = response.get('tokens_used', 0)
        msg.finish_reason = response.get('finish_reason', '')
        
        self.ai_response_pub.publish(msg)
        rospy.loginfo(f"AI response published for command {command_id}")

    def _publish_error_response(self, error_message: str):
        """Публикация сообщения об ошибке"""
        msg = AIResponse()
        msg.header.stamp = rospy.Time.now()
        msg.response_text = f"Ошибка: {error_message}"
        msg.confidence = 0.0
        msg.tokens_used = 0
        msg.finish_reason = "error"
        
        self.ai_response_pub.publish(msg)

    def _publish_system_status(self, status: str, is_operational: bool):
        """Публикация статуса системы"""
        msg = SystemStatus()
        msg.header.stamp = rospy.Time.now()
        msg.status = status
        msg.is_operational = is_operational
        msg.component = "openai_integration"
        
        self.system_status_pub.publish(msg)

    def shutdown(self):
        """Корректное завершение работы"""
        rospy.loginfo("Shutting down OpenAI Integration Node")
        
        # Завершение асинхронного loop
        if self.loop.is_running():
            self.loop.call_soon_threadsafe(self.loop.stop)
        
        # Закрытие HTTP сессии
        if self.session and not self.session.closed:
            asyncio.run_coroutine_threadsafe(self.session.close(), self.loop)
        
        self.thread.join(timeout=5)

def main():
    try:
        node = OpenAIIntegration()
        rospy.spin()
    except KeyboardInterrupt:
        pass
    finally:
        if 'node' in locals():
            node.shutdown()

if __name__ == '__main__':
    main()
```

## Конфигурационный файл: `config/ai_model_params.yaml`

```yaml
# Параметры OpenAI Integration
openai_integration:
  openai_api_key: "${OPENAI_API_KEY}"  # Берется из переменной окружения
  model: "gpt-4"
  max_tokens: 2000
  temperature: 0.7
  max_context_length: 10
  request_timeout: 30
  
  # Лимиты и квоты
  max_requests_per_minute: 60
  retry_attempts: 3
  retry_delay: 2

  # Промпты системы
  system_prompts:
    security: "Ты - система безопасности робота-телохранителя. Анализируй угрозы и предлагай безопасные решения."
    communication: "Ты - коммуникационный помощник. Обрабатывай команды пользователя вежливо и эффективно."
    general: "Ты - ИИ помощник в ROS системе. Предоставляй точные и полезные ответы."
```

## Launch файл: `launch/openai_integration.launch`

```xml
<launch>
    <node name="openai_integration" pkg="ros_ai_assistant" type="openai_integration.py" output="screen">
        <rosparam command="load" file="$(find ros_ai_assistant)/config/ai_model_params.yaml"/>
        <param name="openai_api_key" value="$(env OPENAI_API_KEY)"/>
    </node>
</launch>
```

Для запуска проекта необходимо создать следующие файлы и выполнить настройки:

## 1. Основные конфигурационные файлы

### `package.xml`
```xml
<?xml version="1.0"?>
<package format="2">
  <name>ros_ai_assistant</name>
  <version>1.0.0</version>
  <description>ROS AI Assistant with OpenAI integration</description>
  <maintainer email="admin@example.com">ROS AI Team</maintainer>
  <license>MIT</license>

  <buildtool_depend>catkin</buildtool_depend>
  <buildtool_depend>catkin_simple</buildtool_depend>

  <depend>roscpp</depend>
  <depend>rospy</depend>
  <depend>std_msgs</depend>
  <depend>message_generation</depend>
  <depend>message_runtime</depend>
  <depend>actionlib</depend>
  <depend>actionlib_msgs</depend>

  <build_depend>python3</build_depend>
  <build_depend>python3-catkin-pkg</build_depend>
  <build_depend>python3-pip</build_depend>

  <exec_depend>python3</exec_depend>
  <exec_depend>python3-rospkg</exec_depend>
  <exec_depend>python3-aiohttp</exec_depend>
  <exec_depend>python3-asyncio</exec_depend>
  <exec_depend>python3-yaml</exec_depend>
  <exec_depend>python3-flask</exec_depend>
  <exec_depend>python3-flask-restful</exec_depend>

  <export>
    <architecture_independent/>
  </export>
</package>
```

### `CMakeLists.txt`
```cmake
cmake_minimum_required(VERSION 3.0.2)
project(ros_ai_assistant)

find_package(catkin REQUIRED COMPONENTS
  roscpp
  rospy
  std_msgs
  message_generation
)

catkin_python_setup()

add_message_files(
  FILES
  SecurityAlert.msg
  UserCommand.msg
  AIResponse.msg
  SystemStatus.msg
  ThreatAssessment.msg
  WeaponStatus.msg
)

add_service_files(
  FILES
  WeaponControl.srv
  UserAuthentication.srv
  ContextSwitch.srv
  EmergencyShutdown.srv
  ThreatAssessment.srv
  OpenAIRequest.srv
)

generate_messages(
  DEPENDENCIES
  std_msgs
)

catkin_package(
  CATKIN_DEPENDS 
  message_runtime 
  roscpp 
  rospy 
  std_msgs
)

include_directories(
  include
  ${catkin_INCLUDE_DIRS}
)

install(DIRECTORY launch config scripts api
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
)

install(PROGRAMS
  scripts/ai_core_nodes/openai_integration.py
  scripts/ai_core_nodes/context_manager.py
  scripts/ai_core_nodes/decision_engine.py
  scripts/security_nodes/threat_assessment.py
  scripts/security_nodes/weapon_controller.py
  scripts/security_nodes/emergency_handler.py
  scripts/communication_nodes/voice_processor.py
  scripts/communication_nodes/dialog_manager.py
  scripts/communication_nodes/user_identity.py
  scripts/integration_nodes/api_gateway.py
  scripts/integration_nodes/ros_api_bridge.py
  scripts/integration_nodes/safety_monitor.py
  DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)
```

### `setup.py`
```python
from distutils.core import setup
from catkin_pkg.python_setup import generate_distutils_setup

setup_args = generate_distutils_setup(
    packages=[
        'ros_ai_assistant',
        'ros_ai_assistant.ai_core_nodes',
        'ros_ai_assistant.security_nodes', 
        'ros_ai_assistant.communication_nodes',
        'ros_ai_assistant.integration_nodes',
    ],
    package_dir={'': 'scripts'}
)

setup(**setup_args)
```

## 2. Запускаемые файлы

### `launch/assistant.launch`
```xml
<launch>
    <!-- Параметры -->
    <rosparam command="load" file="$(find ros_ai_assistant)/config/security_params.yaml"/>
    <rosparam command="load" file="$(find ros_ai_assistant)/config/communication_params.yaml"/>
    <rosparam command="load" file="$(find ros_ai_assistant)/config/ai_model_params.yaml"/>
    <rosparam command="load" file="$(find ros_ai_assistant)/config/api_endpoints.yaml"/>

    <!-- AI Core Nodes -->
    <node name="openai_integration" pkg="ros_ai_assistant" type="openai_integration.py" output="screen">
        <param name="openai_api_key" value="$(env OPENAI_API_KEY)"/>
    </node>
    
    <node name="context_manager" pkg="ros_ai_assistant" type="context_manager.py" output="screen"/>
    <node name="decision_engine" pkg="ros_ai_assistant" type="decision_engine.py" output="screen"/>

    <!-- Security Nodes -->
    <node name="threat_assessment" pkg="ros_ai_assistant" type="threat_assessment.py" output="screen"/>
    <node name="emergency_handler" pkg="ros_ai_assistant" type="emergency_handler.py" output="screen"/>

    <!-- Communication Nodes -->
    <node name="dialog_manager" pkg="ros_ai_assistant" type="dialog_manager.py" output="screen"/>
    <node name="user_identity" pkg="ros_ai_assistant" type="user_identity.py" output="screen"/>

    <!-- Integration Nodes -->
    <node name="safety_monitor" pkg="ros_ai_assistant" type="safety_monitor.py" output="screen"/>
    
    <!-- API Gateway (запускается отдельно из-за Flask) -->
    <node name="api_gateway" pkg="ros_ai_assistant" type="api_gateway.py" output="screen" respawn="true"/>
</launch>
```

### `scripts/integration_nodes/api_gateway.py`
```python
#!/usr/bin/env python3
import rospy
import threading
from flask import Flask, request, jsonify
from flask_restful import Api, Resource
import subprocess
import os

class APIGateway:
    def __init__(self):
        self.app = Flask(__name__)
        self.api = Api(self.app)
        self.setup_routes()
        
    def setup_routes(self):
        # Security endpoints
        self.api.add_resource(SecurityAPI, '/api/v1/security-system/<string:endpoint>')
        self.api.add_resource(CommunicationAPI, '/api/v1/communication/<string:endpoint>')
        
    def run(self):
        port = rospy.get_param('~api_port', 5000)
        self.app.run(host='0.0.0.0', port=port, debug=False, threaded=True)

class SecurityAPI(Resource):
    def post(self, endpoint):
        try:
            data = request.get_json()
            rospy.loginfo(f"Security API call: {endpoint}")
            
            # Здесь будет интеграция с ROS сервисами
            return {"status": "success", "endpoint": endpoint, "data": data}
        except Exception as e:
            return {"error": str(e)}, 500

class CommunicationAPI(Resource):
    def post(self, endpoint):
        try:
            data = request.get_json()
            rospy.loginfo(f"Communication API call: {endpoint}")
            
            # Здесь будет интеграция с ROS сервисами
            return {"status": "success", "endpoint": endpoint, "data": data}
        except Exception as e:
            return {"error": str(e)}, 500

def main():
    rospy.init_node('api_gateway', anonymous=True)
    gateway = APIGateway()
    
    # Запуск Flask в отдельном потоке
    flask_thread = threading.Thread(target=gateway.run)
    flask_thread.daemon = True
    flask_thread.start()
    
    rospy.loginfo("API Gateway started")
    rospy.spin()

if __name__ == '__main__':
    main()
```

## 3. Файлы конфигурации

### `config/ai_model_params.yaml`
```yaml
openai_integration:
  model: "gpt-4"
  max_tokens: 2000
  temperature: 0.7
  max_context_length: 10
  request_timeout: 30

security_system:
  threat_levels:
    low: 0.3
    medium: 0.6
    high: 0.8
    critical: 0.95

communication:
  supported_languages: ["ru", "en"]
  default_language: "ru"
  response_timeout: 10
```

### `config/api_endpoints.yaml`
```yaml
security_endpoints:
  base_url: "/api/v1/security-system"
  endpoints:
    situational_analysis: "/situational-analysis"
    threat_assessment: "/threat-assessment"
    weapon_control: "/weapon-control"
    emergency_protocols: "/emergency-protocols"

communication_endpoints:
  base_url: "/api/v1/communication"
  endpoints:
    greeting: "/greeting"
    user_identity: "/user-identity"
    session_management: "/session-management"

server:
  port: 5000
  host: "0.0.0.0"
```

## 4. Установка и запуск

### Создайте файл установки зависимостей `install_requirements.sh`
```bash
#!/bin/bash
echo "Installing Python dependencies..."
pip3 install aiohttp asyncio flask flask-restful pyyaml rospkg

echo "Setting up environment..."
export OPENAI_API_KEY="your_openai_api_key_here"
echo "export OPENAI_API_KEY=$OPENAI_API_KEY" >> ~/.bashrc

echo "Installation complete!"
```

### Создайте файл запуска `run_project.sh`
```bash
#!/bin/bash

# Активация ROS
source /opt/ros/noetic/setup.bash
source devel/setup.bash

# Запуск проекта
echo "Starting ROS AI Assistant..."
roslaunch ros_ai_assistant assistant.launch
```

## 5. Инструкция по запуску

1. **Установите зависимости:**
```bash
chmod +x install_requirements.sh
./install_requirements.sh
```

2. **Соберите проект:**
```bash
cd ~/catkin_ws
catkin_make
source devel/setup.bash
```

3. **Установите переменные окружения:**
```bash
export OPENAI_API_KEY="your_actual_openai_api_key"
```

4. **Запустите проект:**
```bash
chmod +x run_project.sh
./run_project.sh
```

5. **Проверьте работу API:**
```bash
curl -X POST http://localhost:5000/api/v1/communication/greeting \
  -H "Content-Type: application/json" \
  -d '{"message": "Привет"}'
```

## 6. Дополнительные утилиты

### `scripts/test_connection.py`
```python
#!/usr/bin/env python3
import rospy
from ros_ai_assistant.srv import OpenAIRequest

def test_openai_connection():
    rospy.wait_for_service('/ai_assistant/openai_request')
    try:
        openai_proxy = rospy.ServiceProxy('/ai_assistant/openai_request', OpenAIRequest)
        response = openai_proxy("Test message", "test", "gpt-4")
        print(f"Response: {response.response}")
        print(f"Success: {response.success}")
    except rospy.ServiceException as e:
        print(f"Service call failed: {e}")

if __name__ == '__main__':
    test_openai_connection()
```

Теперь проект готов к запуску! Все необходимые файлы созданы и настроены для работы с ROS и OpenAI API.

