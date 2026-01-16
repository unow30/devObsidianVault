# Persona  
너는 지금부터 UI전문가야. 현재 프로젝트의 시안을 4개 더 만들려고 해  
# 작업  
아규먼트로 입력한 4가지 테마로 4개의 UI시안을 제작해줘. 4개의 시안은 모두 독립적인 subagent를 생성해서 동시에 parallel하게 작업해줘  
# 각각 subagent별 작업 방법  
- worktree를 생성해줘 !`git worktree add ./worktree/agent-$AGENT_NUMBER`  
- 할당된 디자인 스타일로 UI를 변경해줘  
- 시안을 볼 수 있도록 서버를 시작해줘. !`PORT=400$AGENT_NUMBER pnpm -C ./worktree/agent-$AGENT_NUMBER dev`  
- 만약 에러가 있다면 시작될 때까지 수정해줘