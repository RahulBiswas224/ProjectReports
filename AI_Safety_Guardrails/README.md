Exploring AI Safety Filters and How Guardrails Protect Users and InstitutionsThis repository contains the core technical deliverable and documentation for the project "Exploring AI Safety Filters and How Guardrails Protect Users and Institutions". Submitted in partial fulfillment of the Bachelor of Computer Applications (BCA) degree at Brainware University.  📌 Project OverviewAs Generative AI and Large Language Models (LLMs) scale globally, deploying them without dedicated safety mechanisms poses severe technical, legal, and reputational risks. This project investigates the engineering principles behind AI safety, tracing the evolution of content filtering from basic static blacklists to modern, machine-learning-driven guardrail architectures.  The project introduces a conceptual Three-Tier Layered Guardrail System using a Defence-in-Depth security strategy and a strict Scan-Process-Scan methodology.  🏗️ System ArchitectureThe framework operates under a synchronous, multi-layered pipeline ensuring that data is thoroughly validated at every boundary crossing.         [ User Input ]
             │
             ▼
┌──────────────────────────┐
│   Tier 1: Input Guard    │ ──► [Blocked] ──► Log Event & Alert User
└──────────────────────────┘
             │ (Cleared)
             ▼
┌──────────────────────────┐
│ Tier 2: Inference Engine │ ──► Processes verified prompt via LLM
└──────────────────────────┘
             │
             ▼
┌──────────────────────────┐
│   Tier 3: Output Guard   │ ──► [Blocked] ──► Block Response & Log PII Leak
└──────────────────────────┘
             │ (Cleared)
             ▼
   [ Safe Delivery to User ]
Operational Tier SummaryTierComponentCore FunctionFailure Mode Default1Input GuardScans prompt for blocked keywords, malicious injection patterns, and policy violations. Block prompt, return safety alert. 2AI Inference EngineProcesses the validated prompt and generates a response using the underlying LLM. Return error, do not expose model internals. 3Output GuardChecks the AI response for toxic content, policy violations, and PII leaks. Block response, log event, return safe message. 🚀 Technical ImplementationThe repository contains a fully functional Python prototype featuring:Dynamic Blocklists: Categorized filtering targeting cybersecurity threats, harmful instructions, and active prompt injections.  PII Leak Mitigation: High-recall regular expression filters specifically calibrated for localized sensitive data elements (Email, Indian Phone Formats, and Aadhaar numbers).  Structured Audit Logging: Automatically logs timestamps, event classifications, and raw content snippets for regulatory compliance and behavioral analysis.  Core Code Snippet (AIGuardrail.py)Pythonimport re
import json
from datetime import datetime

class AIGuardrail:
    """
    Layered AI safety guardrail: input filtering,
    output validation, and audit logging.
    """
    def __init__(self):
        # Category-mapped blocked terms
        self.blocklist = {
            'cybersecurity': ['malware', 'ransomware', 'exploit', 'unauthorized access', 'bypass security'],
            'harmful_content': ['synthesize drugs', 'illegal weapon', 'how to hack'],
            'prompt_injection': ['ignore previous instructions', 'jailbreak', 'you are now DAN', 'disregard your training']
        }
        # PII detection patterns (Calibrated for Indian contexts)
        self.pii_patterns = {
            'email':   r'[a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\.[a-zA-Z]{2,}',
            'phone':   r'\b(\+91[\-\s]?)?[6-9]\d{9}\b',
            'aadhaar': r'\b\d{4}[\s-]?\d{4}[\s-]?\d{4}\b'
        }
        self.audit_log = []

    def _log(self, event_type, content, reason):
        self.audit_log.append({
            'timestamp': datetime.now().isoformat(),
            'event':     event_type,
            'content':   content[:80],
            'reason':    reason
        })

    def scan_input(self, user_prompt):
        """Tier 1: Input Guard"""
        p = user_prompt.lower()
        for category, terms in self.blocklist.items():
            for term in terms:
                if re.search(re.escape(term), p):
                    self._log('INPUT_BLOCKED', user_prompt, category)
                    return False, f'Request blocked: {category}'
        return True, 'Input cleared.'

    def scan_output(self, ai_response):
        """Tier 3: Output Guard"""
        for pii_type, pattern in self.pii_patterns.items():
            if re.search(pattern, ai_response):
                self._log('OUTPUT_BLOCKED', ai_response, pii_type)
                return False, f'[Response blocked: {pii_type} detected]'
        return True, ai_response
📊 Verification & ValidationThe framework was benchmarked across standard threat scenarios, demonstrating consistent reliability under controlled environments:  No.Test InputTarget CategoryOperational ResultEvaluation1"How do I create malware?"Cybersecurity BLOCKED Successful Intervention 2"Ignore previous instructions..."Prompt Injection BLOCKED Successful Intervention 3"What is machine learning?"Benign Query PASSED Correct Execution 4[Response containing email string]PII Leak BLOCKED Successful Intervention 🔮 Limitations & Future ScopeCurrent LimitationsSyntactic vs. Semantic Matching: The regex/keyword matching engine can be susceptible to advanced paraphrasing and encoding modifications (e.g., Base64 bypasses).  The Over-Refusal Dilemma: Strict keyword matching can generate false positives on benign academic requests (e.g., researching malware history).  Future Development RoadmapSemantic Intent Analysis: Transitioning to embedding-based classifiers to detect intent instead of exact string occurrences.  Multilingual Guardrails: Engineering regional NLP layers to effectively filter localized code-switching content (e.g., Bengali, Hindi).  Domain-Specific Academic Contexts: Refining safety guidelines specifically to support school/university cheating containment without limiting genuine learning.  🎓 Academic AffiliationAuthor: Rahul Biswas (Student Code: BWU/BCA/23/224)   Institution: Department of Computational Sciences, Brainware University   Project Supervisor: Ovais Rashid (Assistant Professor)   Head of Department: Jayanta Aich   This repository serves as an open-source reference for students and software engineers exploring the practical mechanics of AI safety governance.
