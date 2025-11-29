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

