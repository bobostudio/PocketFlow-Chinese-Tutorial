# PocketFlow之**单智能体**(Agent)-旅行规划助手

本小节，您将了解如何实现一个单 Agent，单 Agent 模式是指使用一个智能体（Agent）来完成一系列任务的模式，在 PocketFlow 中，这个 Agent 由多个节点（Nodes）组成，这些节点通过Flow 连接起来，形成一个决策和执行流程。

## 项目流程图

graph TD
    A[DecideAction] -->|"search"| B[SearchWeb]
    A -->|"answer"| C[AnswerQuestion]
    B -->|"decide"| A

## 前置条件: [环境安装与配置](./init-env.md)

## 步骤1: 创建项目目录结构

```bash
# 进入虚拟环境
cd p3
# 进入项目目录
cd workspace
# 创建项目文件
touch main4.py flows/travelFlow.py
```

## 步骤2: 定义行动决定节点

在 `travelFlow.py` 中，添加以下代码:

```python
from pocketflow import Node,Flow
from utils.call_llm import call_llm,search_web
import yaml

class DecideAction(Node):
    def prep(self,shared):
        context = shared.get("context","没有之前的搜索")
        travel_query = shared['travel_query']
        return travel_query,context
    def exec(self,inputs):
        travel_query,context = inputs
        prompt = f"""
### 上下文  
你是一个旅行规划助手，可以搜索网络获取信息。  
旅行查询: {travel_query}  
已有信息: {context}  
  
### 可用行动  
[1] search  
  描述: 在网络上查找更多信息  
  参数:  
    - query (str): 要搜索的内容  
  
[2] answer  
  描述: 用当前知识回答问题  
  参数:  
    - answer (str): 给出的旅行建议  
  
## 下一步行动  
根据上下文和可用行动决定下一步。  
按以下格式返回:  
  
```yaml  
thinking: |  
    <你的逐步思考过程>  
action: search 或 answer  
reason: <为什么选择这个行动>  
answer: <如果行动是answer>
search_query: <如果行动是search，具体的搜索查询>
```"""

        res = call_llm(prompt)
        yaml_str = res.split("```yaml")[1].split("```")[0].strip()
        return yaml.safe_load(yaml_str)
    
    def post(self,shared,prep_str,exec_str):
        if exec_str['action'] == "search":
            shared['search_query'] = exec_str['search_query']
            #print(f"🔍 旅行助手决定搜索: {exec_str['search_query']}")  
        else:
            shared['context'] = exec_str['answer']
            #print(f"💭 旅行助手的回答: {exec_str['answer']}")
        
        return exec_str["action"]

```

## 步骤3: 定义网络搜索节点

在 `weatherFlow.py` 中，紧接着添加以下代码:

```python
class SearchWeb(Node):
    def prep(self,shared):
        return shared['search_query']
    def exec(self,search_query):
        print(f"🌐 正在搜索: {search_query}")  
        res = search_web(search_query)
        return res
    def post(self,shared,prep_str,exec_str):
        previous = shared.get("context","")
        shared['context'] =  previous + "\n\n搜索: " + shared["search_query"] + "\n结果: " + exec_str 
        print(f"📚 找到信息，正在分析结果...")  
        return "decide"
```

在 `utils/call_llm.py` 中，添加以下代码:

[tavily api key 申请](https://app.tavily.com/home)

```python
from tavily import TavilyClient

tavily_client = TavilyClient(api_key="自己的tavily api key")

def search_web(query):
    res = tavily_client.search(query)
    results_str = "\n\n".join([
        f"标题: {r['title']}\n网址: {r['url']}\n摘要: {r['content']}"
        for r in res['results']
    ])
    return results_str
```


## 步骤4: 定义回答问题节点

在 `weatherFlow.py` 中，紧接着添加以下代码:

```python
class AnswerQuestion(Node):
    def prep(self,shared):
        return shared['travel_query'],shared.get("context","")
    
    def exec(self,inputs):
        travel_query,context = inputs
        print(f"✍️ 正在制定旅行建议...")  
        prompt = f"""  
### 上下文  
基于以下信息，回答旅行查询。  
旅行查询: {travel_query}  
搜集的信息: {context}  
  
## 你的建议:  
提供一个全面的旅行建议，包括:  
1. 目的地概览  
2. 推荐的景点  
3. 最佳旅行时间  
4. 交通和住宿建议  
5. 预算考虑  
"""     
        res = call_llm(prompt)  
        return res
    def post(self,shared,prep_str,exec_str):
        shared['answer'] = exec_str
        print(f"✅ 旅行建议生成成功")  
        return "done"
```


## 步骤5: 定义流程

在 `weatherFlow.py` 中，紧接着添加以下代码:

```python
def create_travel_agent_flow():
    decide_node = DecideAction()
    search_node = SearchWeb()
    answer_node = AnswerQuestion()

    decide_node  - "search" >> search_node
    
    decide_node  - "answer" >> answer_node

    search_node  - "decide" >> decide_node

    return Flow(start=decide_node)
```

## 步骤6: 定义主程序

在 `main4.py` 中，添加以下代码:

```python
from flows.travelFlow import create_travel_agent_flow

def main():
    shared = {
        "travel_query": "我想去日本旅行，帮我规划一下"
    }
    travel_agent_flow = create_travel_agent_flow()
    travel_agent_flow.run(shared)
    print("\n🎯 旅行建议:")  
    print(shared.get("answer", "没有找到建议"))

if __name__ == '__main__':
    main()
```


## 步骤7: 运行程序

```bash
python main4.py
```

## 运行结果

![运行结果](/images/lesson4/1.png)
