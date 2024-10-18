import { useState } from 'react'
import { Input } from "@/components/ui/input"
import { Label } from "@/components/ui/label"
import { Button } from "@/components/ui/button"
import { Textarea } from "@/components/ui/textarea"
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from "@/components/ui/select"
import { PlusCircle, Trash2, FileDown } from 'lucide-react'
import * as XLSX from 'xlsx'
import { jsPDF } from 'jspdf'
import 'jspdf-autotable'

interface JobDetails {
  title: string;
  reason: string;
  hires: string;
  requirements: {
    essential: string;
    preferred: string;
    other: string;
  };
  hireScenario: {
    success: string;
    failure: string;
  };
  budget: string;
  status: 'open' | 'filled' | 'closed';
}

interface RecruitmentProblem {
  id: string;
  description: string;
}

export default function Component() {
  const [jobDetails, setJobDetails] = useState<JobDetails[]>([{
    title: '',
    reason: '',
    hires: '',
    requirements: {
      essential: '',
      preferred: '',
      other: '',
    },
    hireScenario: {
      success: '',
      failure: '',
    },
    budget: '',
    status: 'open',
  }])
  const [deadline, setDeadline] = useState('')
  const [recruitmentProblems, setRecruitmentProblems] = useState<RecruitmentProblem[]>([
    { id: '1', description: '' }
  ])

  const addJobDetail = () => {
    setJobDetails([...jobDetails, {
      title: '',
      reason: '',
      hires: '',
      requirements: {
        essential: '',
        preferred: '',
        other: '',
      },
      hireScenario: {
        success: '',
        failure: '',
      },
      budget: '',
      status: 'open',
    }])
  }

  const removeJobDetail = (index: number) => {
    setJobDetails(jobDetails.filter((_, i) => i !== index))
  }

  const updateJobDetail = (index: number, field: keyof JobDetails, value: string | object) => {
    const newJobDetails = [...jobDetails]
    if (field === 'requirements' || field === 'hireScenario') {
      newJobDetails[index][field] = { ...newJobDetails[index][field], ...value }
    } else {
      newJobDetails[index][field] = value as string
    }
    setJobDetails(newJobDetails)
  }

  const addRecruitmentProblem = () => {
    setRecruitmentProblems([...recruitmentProblems, { id: Date.now().toString(), description: '' }])
  }

  const removeRecruitmentProblem = (id: string) => {
    setRecruitmentProblems(recruitmentProblems.filter(problem => problem.id !== id))
  }

  const updateRecruitmentProblem = (id: string, description: string) => {
    setRecruitmentProblems(recruitmentProblems.map(problem => 
      problem.id === id ? { ...problem, description } : problem
    ))
  }

  const exportToExcel = () => {
    const ws = XLSX.utils.json_to_sheet(jobDetails.map(job => ({
      '職種名': job.title,
      '採用理由': job.reason,
      '採用人数': job.hires,
      '必須条件': job.requirements.essential,
      '歓迎条件': job.requirements.preferred,
      'その他条件': job.requirements.other,
      '採用成功時': job.hireScenario.success,
      '採用失敗時': job.hireScenario.failure,
      '採用予算': job.budget,
      'ステータス': job.status
    })))
    const wb = XLSX.utils.book_new()
    XLSX.utils.book_append_sheet(wb, ws, '採用詳細')

    const problemsWs = XLSX.utils.json_to_sheet(recruitmentProblems.map(problem => ({
      '採用活動での課題': problem.description
    })))
    XLSX.utils.book_append_sheet(wb, problemsWs, '採用活動での課題')

    XLSX.writeFile(wb, '採用計画.xlsx')
  }

  const exportToPDF = () => {
    const doc = new jsPDF()
    doc.text('採用計画', 14, 15)
    doc.setFontSize(12)

    let yPosition = 25
    const pageHeight = doc.internal.pageSize.height

    jobDetails.forEach((job, index) => {
      if (yPosition + 45 > pageHeight) {
        doc.addPage()
        yPosition = 25
      }

      const splitTitle = doc.splitTextToSize(`職種 ${index + 1}: ${job.title}`, 180)
      doc.text(splitTitle, 14, yPosition)
      doc.text(`採用理由: ${job.reason}`, 14, yPosition + 5)
      doc.text(`採用人数: ${job.hires}`, 14, yPosition + 10)
      doc.text(`必須条件: ${job.requirements.essential}`, 14, yPosition + 15)
      doc.text(`歓迎条件: ${job.requirements.preferred}`, 14, yPosition + 20)
      doc.text(`その他条件: ${job.requirements.other}`, 14, yPosition + 25)
      doc.text(`採用成功時: ${job.hireScenario.success}`, 14, yPosition + 30)
      doc.text(`採用失敗時: ${job.hireScenario.failure}`, 14, yPosition + 35)
      doc.text(`採用予算: ${job.budget}`, 14, yPosition + 40)
      doc.text(`ステータス: ${job.status}`, 14, yPosition + 45)

      yPosition += 50
    })

    if (yPosition + 10 > pageHeight) {
      doc.addPage()
      yPosition = 25
    }

    doc.text(`納期: ${deadline}`, 14, yPosition)

    if (yPosition + 20 > pageHeight) {
      doc.addPage()
      yPosition = 25
    }

    doc.text('採用活動での課題:', 14, yPosition + 10)
    recruitmentProblems.forEach((problem, index) => {
      if (yPosition + 15 > pageHeight) {
        doc.addPage()
        yPosition = 25
      }
      doc.text(`${index + 1}. ${problem.description}`, 14, yPosition + 20 + (index * 10))
    })

    doc.save('採用計画.pdf')
  }

  return (
    <div className="max-w-3xl mx-auto p-4 bg-white shadow rounded-lg">
      <h2 className="text-xl font-bold mb-4 text-blue-600 border-b pb-2">貴社の採用状況について</h2>
      <p className="mb-4 text-sm">今回のご提案にあたり、貴社の採用要件について整理をいたしました</p>
      
      <div className="space-y-6">
        {jobDetails.map((job, index) => (
          <div key={index} className="border rounded-lg p-4 space-y-4">
            <div className="flex items-center justify-between">
              <h3 className="text-lg font-semibold">職種 {index + 1}</h3>
              {index > 0 && (
                <Button
                  variant="destructive"
                  size="sm"
                  onClick={() => removeJobDetail(index)}
                >
                  <Trash2 className="h-4 w-4 mr-2" />
                  削除
                </Button>
              )}
            </div>

            <div className="space-y-2">
              <Label htmlFor={`title-${index}`}>職種名</Label>
              <Input
                id={`title-${index}`}
                value={job.title}
                onChange={(e) => updateJobDetail(index, 'title', e.target.value)}
                placeholder="職種名を入力"
              />
            </div>

            <div className="space-y-2">
              <Label htmlFor={`reason-${index}`}>採用理由（背景）</Label>
              <Textarea
                id={`reason-${index}`}
                value={job.reason}
                onChange={(e) => updateJobDetail(index, 'reason', e.target.value)}
                placeholder="採用の理由や背景を入力"
              />
            </div>

            <div className="space-y-2">
              <Label htmlFor={`hires-${index}`}>採用人数</Label>
              <Input
                id={`hires-${index}`}
                type="number"
                value={job.hires}
                onChange={(e) => updateJobDetail(index, 'hires', e.target.value)}
                placeholder="採用予定人数"
              />
            </div>

            <div className="space-y-2">
              <Label htmlFor={`status-${index}`}>ステータス</Label>
              <Select
                value={job.status}
                onValueChange={(value) => updateJobDetail(index
