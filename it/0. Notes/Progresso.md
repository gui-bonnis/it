**SELECT**  
   33386 **AS** _total_,  
**COUNT**(1) **AS** _total_processados_,  
**COUNT**(**CASE** **WHEN** _sit_.STATUS = 'SUCESSO' **THEN** 1 **END**) **AS** _total_sucesso_,  
**COUNT**(**CASE** **WHEN** _sit_.STATUS = 'ERRO' **THEN** 1 **END**) **AS** _total_erro_,  
**ROUND**(**COUNT**(**CASE** **WHEN** _sit_.STATUS = 'SUCESSO' **THEN** 1 **END**) * 100.0 / **COUNT**(1), 2) **AS** _percentual_sucesso_,  
**ROUND**(**COUNT**(**CASE** **WHEN** _sit_.STATUS = 'ERRO' **THEN** 1 **END**) * 100.0 / **COUNT**(1), 2) **AS** _percentual_erro_,  
**ROUND**(**COUNT**(1) * 100.0 / 33386, 2) **AS** _percentual_carga_realizada_,  
**MIN**(_sit_.DATA_HORA) **AS** _horario_inicio_,  
-- Calcula o tempo médio por item processado  
**CASE**  
**WHEN** **COUNT**(1) > 1 **THEN**  
(**MAX**(_sit_.DATA_HORA) - **MIN**(_sit_.DATA_HORA)) / (**COUNT**(1) - 1)  
**ELSE**  
**INTERVAL** '0' **MINUTE**  
**END** **AS** _tempo_medio_por_item_,  
-- Projeta o horário de fim com base no tempo médio por item  
**CASE**  
**WHEN** **COUNT**(1) > 1 **THEN**  
**MAX**(_sit_.DATA_HORA) + ((33386 - **COUNT**(1)) *  
(**MAX**(_sit_.DATA_HORA) - **MIN**(_sit_.DATA_HORA)) / (**COUNT**(1) - 1))  
**ELSE**  
**MAX**(_sit_.DATA_HORA)  
**END** **AS** _horario_projetado_fim_  
**FROM**  
TOTVS.STATUS_INTEGRACAO_ALUNO _sit_  
**INNER** **JOIN** ACD.FILIAL f  
   **ON** _sit_.COD_INST = _f_.COD_INST  
**WHERE**  
_sit_.DATA_HORA > **TO_DATE**('2024-11-26 08:00:00', 'yyyy-mm-dd hh24:mi:ss')  
**AND** f.COLI_RM **IN** (4,5,18)  
**AND** _sit_.ID_STATUS_INTEGRACAO_ALUNO = (  
       **SELECT** **MAX**(_sit2_.ID_STATUS_INTEGRACAO_ALUNO)  
       **FROM** TOTVS.STATUS_INTEGRACAO_ALUNO _sit2_  
       **WHERE** _sit_.COD_INST = _sit2_.COD_INST **AND** _sit_.RGM_ALUN = _sit2_.RGM_ALUN  
   );