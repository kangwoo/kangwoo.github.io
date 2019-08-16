---
title:  "Istio client go"
classes: wide
date: 2019-08-01T20:22:10+09:00
categories: [devops, kubernetes]
tags: [kubernetes, istio]
---

## 시작하기전에..
애플리케이션에서 Istio CR(Custom Resources)을 생성해야 하는데, 공식적으로 제공하는 라이브러리가 없다.

구글에 검색해 본 결과 [istio-client-go](https://github.com/aspenmesh/istio-client-go)가 존재한다.
하지만, 필요한 리소그 몇개가 빠져 있어어서 재미삼아 만들어봤다.

주소 : https://github.com/kangwoo/istio-client-go

## 준비물
처음에는 `kubebuilder`를 사용하려 했으나, 현재 버전(2.0.0-beta.0)에서는 복수개의 그룹을 지원하지 않는다. (Multiple groups are not supported yet)
그래서 `operator-sdk`를 사용한다.
 - golang
 - operator-sdk

## 프로젝트 생성
`operator-sdk`의 `new` 명령어를 사용해서 프로젝트 생성한다.
```bash
$ operator-sdk new istio-client-go --repo github.com/kangwoo/istio-client-go
$ cd istio-client-go 
```

## istio 추가
```bash
$ go get istio.io/api
```

## 필요한 리소스 추가
```bash
$ operator-sdk add api --api-version=authentication.istio.io/v1alpha1 --kind=Policy
$ operator-sdk add api --api-version=networking.istio.io/v1alpha3 --kind=Gateway
$ operator-sdk add api --api-version=rbac.istio.io/v1alpha1 --kind=ServiceRole
$ operator-sdk add api --api-version=rbac.istio.io/v1alpha1 --kind=ServiceRoleBinding
```

## 불필요한 파일 삭제 및 코드 수정
- PolicySpec 타입 위의 `// +k8s:openapi-gen=true` 제거
- PolicyStatus 타입 제거 및 Policy 타입에 Status 제거
- json 변환 및 deepcopy 구현
    ```
    import (
        "bufio"
        "bytes"
        "log"
    
        "github.com/gogo/protobuf/jsonpb"
        metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
        istiov1alpha1 "istio.io/api/authentication/v1alpha1"
    )
    
    ...
    
        
    func (p *PolicySpec) MarshalJSON() ([]byte, error) {
        buffer := bytes.Buffer{}
        writer := bufio.NewWriter(&buffer)
        marshaler := jsonpb.Marshaler{}
        err := marshaler.Marshal(writer, &p.Policy)
        if err != nil {
            log.Printf("Could not marshal PolicySpec. Error: %v", err)
            return nil, err
        }
    
        writer.Flush()
        return buffer.Bytes(), nil
    }
    
    func (p *PolicySpec) UnmarshalJSON(b []byte) error {
        reader := bytes.NewReader(b)
        unmarshaler := jsonpb.Unmarshaler{}
        err := unmarshaler.Unmarshal(reader, &p.Policy)
        if err != nil {
            log.Printf("Could not unmarshal PolicySpec. Error: %v", err)
            return err
        }
        return nil
    }
    
    // DeepCopyInto is a deepcopy function, copying the receiver, writing into out. in must be non-nil.
    // Based of https://github.com/istio/istio/blob/release-0.8/pilot/pkg/config/kube/crd/types.go#L450
    func (in *PolicySpec) DeepCopyInto(out *PolicySpec) {
        *out = *in
    }
    
    ```

## 참고 코드
- 원본 파일
    ```golang
    package v1alpha1
    
    import (
        metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
    )
    
    // EDIT THIS FILE!  THIS IS SCAFFOLDING FOR YOU TO OWN!
    // NOTE: json tags are required.  Any new fields you add must have json tags for the fields to be serialized.
    
    // PolicySpec defines the desired state of Policy
    // +k8s:openapi-gen=true
    type PolicySpec struct {
        // INSERT ADDITIONAL SPEC FIELDS - desired state of cluster
        // Important: Run "operator-sdk generate k8s" to regenerate code after modifying this file
        // Add custom validation using kubebuilder tags: https://book-v1.book.kubebuilder.io/beyond_basics/generating_crd.html
    }
    
    // PolicyStatus defines the observed state of Policy
    // +k8s:openapi-gen=true
    type PolicyStatus struct {
        // INSERT ADDITIONAL STATUS FIELD - define observed state of cluster
        // Important: Run "operator-sdk generate k8s" to regenerate code after modifying this file
        // Add custom validation using kubebuilder tags: https://book-v1.book.kubebuilder.io/beyond_basics/generating_crd.html
    }
    
    // +k8s:deepcopy-gen:interfaces=k8s.io/apimachinery/pkg/runtime.Object
    
    // Policy is the Schema for the policies API
    // +k8s:openapi-gen=true
    // +kubebuilder:subresource:status
    type Policy struct {
        metav1.TypeMeta   `json:",inline"`
        metav1.ObjectMeta `json:"metadata,omitempty"`
    
        Spec   PolicySpec   `json:"spec,omitempty"`
        Status PolicyStatus `json:"status,omitempty"`
    }
    
    // +k8s:deepcopy-gen:interfaces=k8s.io/apimachinery/pkg/runtime.Object
    
    // PolicyList contains a list of Policy
    type PolicyList struct {
        metav1.TypeMeta `json:",inline"`
        metav1.ListMeta `json:"metadata,omitempty"`
        Items           []Policy `json:"items"`
    }
    
    func init() {
        SchemeBuilder.Register(&Policy{}, &PolicyList{})
    }
    ```

- 수정 후 파일
    ```
    package v1alpha1
    
    import (
        "bufio"
        "bytes"
        "log"
    
        "github.com/gogo/protobuf/jsonpb"
        metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
        istiov1alpha1 "istio.io/api/authentication/v1alpha1"
    )
    
    // EDIT THIS FILE!  THIS IS SCAFFOLDING FOR YOU TO OWN!
    // NOTE: json tags are required.  Any new fields you add must have json tags for the fields to be serialized.
    
    // PolicySpec defines the desired state of Policy
    type PolicySpec struct {
        // INSERT ADDITIONAL SPEC FIELDS - desired state of cluster
        // Important: Run "operator-sdk generate k8s" to regenerate code after modifying this file
        // Add custom validation using kubebuilder tags: https://book-v1.book.kubebuilder.io/beyond_basics/generating_crd.html
        istiov1alpha1.Policy
    }
    
    
    // +k8s:deepcopy-gen:interfaces=k8s.io/apimachinery/pkg/runtime.Object
    
    // Policy is the Schema for the policies API
    // +k8s:openapi-gen=true
    // +kubebuilder:subresource:status
    type Policy struct {
        metav1.TypeMeta   `json:",inline"`
        metav1.ObjectMeta `json:"metadata,omitempty"`
    
        Spec   PolicySpec   `json:"spec,omitempty"`
    }
    
    // +k8s:deepcopy-gen:interfaces=k8s.io/apimachinery/pkg/runtime.Object
    
    // PolicyList contains a list of Policy
    type PolicyList struct {
        metav1.TypeMeta `json:",inline"`
        metav1.ListMeta `json:"metadata,omitempty"`
        Items           []Policy `json:"items"`
    }
    
    func init() {
        SchemeBuilder.Register(&Policy{}, &PolicyList{})
    }
    
    func (p *PolicySpec) MarshalJSON() ([]byte, error) {
        buffer := bytes.Buffer{}
        writer := bufio.NewWriter(&buffer)
        marshaler := jsonpb.Marshaler{}
        err := marshaler.Marshal(writer, &p.Policy)
        if err != nil {
            log.Printf("Could not marshal PolicySpec. Error: %v", err)
            return nil, err
        }
    
        writer.Flush()
        return buffer.Bytes(), nil
    }
    
    func (p *PolicySpec) UnmarshalJSON(b []byte) error {
        reader := bytes.NewReader(b)
        unmarshaler := jsonpb.Unmarshaler{}
        err := unmarshaler.Unmarshal(reader, &p.Policy)
        if err != nil {
            log.Printf("Could not unmarshal PolicySpec. Error: %v", err)
            return err
        }
        return nil
    }
    
    // DeepCopyInto is a deepcopy function, copying the receiver, writing into out. in must be non-nil.
    // Based of https://github.com/istio/istio/blob/release-0.8/pilot/pkg/config/kube/crd/types.go#L450
    func (in *PolicySpec) DeepCopyInto(out *PolicySpec) {
        *out = *in
    }
    ```

 코드 생성
```bash
$ operator-sdk generate k8s
```
