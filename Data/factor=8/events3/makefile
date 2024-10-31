all : clean


clean:
	-(! (kubectl get namespaces -o custom-columns=":metadata.name"|grep sim)) ||kubectl delete ns $$(kubectl get namespaces -o custom-columns=":metadata.name"|grep sim)
	-(! kubectl delete pv $$(kubectl get pv -o custom-columns=:metadata.name))
lunch:
	python3 simulation.py
